import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

st.set_page_config(page_title="Apple of Fortune Stats", page_icon="🍏", layout="centered")

st.title("🍏 Analyseur Apple of Fortune")
st.markdown("**Importe ton fichier CSV pour voir tes statistiques de jeu.**")

# Charger le fichier
uploaded_file = st.file_uploader("📂 Choisis ton fichier CSV :", type=["csv"])

if uploaded_file is not None:
    df = pd.read_csv(uploaded_file)
    st.subheader("Aperçu des données :")
    st.dataframe(df.head())

    # Nettoyage
    df['result'] = df['result'].str.lower().str.strip()
    df['bet'] = pd.to_numeric(df['bet'], errors='coerce')
    df['payout'] = pd.to_numeric(df['payout'], errors='coerce')

    total_games = len(df)
    wins = (df['result'] == 'win').sum()
    losses = (df['result'] == 'lose').sum()
    win_rate = (wins / total_games) * 100
    total_payout = df['payout'].sum()
    total_bet = df['bet'].sum()
    rtp = (total_payout / total_bet) * 100
    gain_moyen = total_payout / total_games
    rentabilite = total_payout / total_bet

    st.subheader("📊 Statistiques globales")
    st.write(f"**Total parties :** {total_games}")
    st.write(f"**Victoires :** {wins} | **Défaites :** {losses}")
    st.write(f"**Taux de victoire :** {win_rate:.2f}%")
    st.write(f"**RTP estimé :** {rtp:.2f}%")
    st.write(f"**Gain moyen par partie :** {gain_moyen:.3f}")
    st.write(f"**Rentabilité par mise :** {rentabilite:.3f}")

    # Analyse par niveau
    stats_par_niveau = df.groupby('level').agg(
        parties=('level', 'count'),
        victoires=('result', lambda x: (x == 'win').sum()),
        moyenne_gain=('payout', 'mean'),
        taux_victoire=('result', lambda x: (x == 'win').mean() * 100)
    ).reset_index()

    st.subheader("📈 Statistiques par niveau")
    st.dataframe(stats_par_niveau)

    # Graphique
    st.subheader("📊 Graphique du taux de victoire par niveau")
    fig, ax = plt.subplots()
    ax.bar(stats_par_niveau['level'], stats_par_niveau['taux_victoire'])
    ax.set_xlabel("Niveau")
    ax.set_ylabel("Taux de victoire (%)")
    ax.set_title("Taux de victoire par niveau")
    st.pyplot(fig)

    # Export Excel
    stats_par_niveau.to_excel("apple_of_fortune_stats.xlsx", index=False)
    with open("apple_of_fortune_stats.xlsx", "rb") as file:
        st.download_button(
            label="📥 Télécharger les stats en Excel",
            data=file,
            file_name="apple_of_fortune_stats.xlsx",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        )
else:
    st.info("⬆️ Charge ton fichier CSV pour commencer.")
