import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import os
from datetime import date

st.set_page_config(page_title="App de Treino", layout="wide")
st.title("💪 App de Treino - Frederico")

ARQUIVO = "historico_treinos.csv"

# Base de exercícios
treinos = {
    "Treino A - Posterior/Glúteos": ["Banco Sóleo", "Panturrilha Guiada", "Cadeira Extensora", "Cadeira Flexora Uni", "Mesa Flexora", "Agachamento Sumô", "Stiff", "Cadeira Abdutora", "Leg 45 Uni"],
    "Treino B - Dorsal": ["Abdominal supra", "Pullover máquina", "Puxada Artic. Uni", "Remada Uni TANKER", "Remada CIMERIAN", "Cavalinho Aberto", "Levantamento Terra", "Hiperextensão lombar"],
    "Treino C - Ombros": ["Banco Sóleo", "Panturrilha Guiada", "Elev. lateral halteres", "Elev. lateral IPIRANGA", "Desenv. SMITH", "Elev. Frontal CROSS", "Encolhimento", "Fly Inverso"]
}

tab1, tab2 = st.tabs(["📝 Registar Treino", "📊 Dashboard e Evolução"])

with tab1:
    escolha = st.selectbox("Qual treino vai fazer hoje?", list(treinos.keys()))
    data_hoje = st.date_input("Data", date.today())
    
    st.write(f"**Preencha as cargas para o {escolha}:**")
    resultados = []
    
    with st.form("form_treino"):
        for exercicio in treinos[escolha]:
            col1, col2 = st.columns([3, 1])
            with col1:
                carga = st.number_input(f"{exercicio} - Carga (kg)", min_value=0, step=1)
            with col2:
                status = st.selectbox(f"Status ({exercicio})", ["OK", "NOK"])
            resultados.append({"Data": data_hoje, "Treino": escolha, "Exercicio": exercicio, "Carga": carga, "Status": status})
            st.divider()
        
        salvar = st.form_submit_button("💾 Salvar Treino")
        
        if salvar:
            df_novo = pd.DataFrame(resultados)
            if os.path.exists(ARQUIVO):
                df_antigo = pd.read_csv(ARQUIVO)
                df_final = pd.concat([df_antigo, df_novo])
            else:
                df_final = df_novo
            df_final.to_csv(ARQUIVO, index=False)
            st.success("Treino salvo com sucesso! Vá à aba Dashboard para ver a evolução.")

with tab2:
    if os.path.exists(ARQUIVO):
        df = pd.read_csv(ARQUIVO)
        st.write("### O seu Histórico")
        st.dataframe(df)
        
        st.write("### Evolução de Cargas")
        exercicio_alvo = st.selectbox("Escolha o exercício para ver o gráfico:", df["Exercicio"].unique())
        
        df_filtro = df[df["Exercicio"] == exercicio_alvo]
        if not df_filtro.empty:
            fig, ax = plt.subplots(figsize=(8, 4))
            ax.plot(df_filtro["Data"], df_filtro["Carga"], marker='o', color='#00ffcc', linewidth=2)
            ax.set_title(f"Evolução: {exercicio_alvo}", color="white")
            ax.set_ylabel("Carga (kg)", color="white")
            ax.tick_params(colors='white')
            fig.patch.set_facecolor('#0e1117')
            ax.set_facecolor('#0e1117')
            st.pyplot(fig)
    else:
        st.warning("Nenhum treino registado ainda. Salve o seu primeiro treino para gerar o Dashboard.")
