import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

data = {
    'id_venda': [1, 2, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20],
    'produto': ['Arroz 5kg', 'Feijão', 'Feijão', 'Leite', 'Arroz 5kg', 'Óleo', 'Pão', 'Leite', 'Café', 'Açúcar', 
                'Arroz 5kg', 'Biscoito', 'Óleo', 'Pão', 'Café', 'Leite', 'Manteiga', 'Açúcar', 'Arroz 5kg', 'Feijão', 'Manteiga'],
    'categoria': ['Grãos', 'Grãos', 'Grãos', 'Laticínio', 'Grãos', 'Mercearia', 'Padaria', 'Laticínio', 'Mercearia', 'Mercearia',
                  'Grãos', 'Biscoito', 'Mercearia', 'Padaria', 'Mercearia', 'Laticínio', 'Laticínio', 'Mercearia', 'Grãos', 'Grãos', 'Laticínio'],
    'valor_venda': [30.00, 8.50, 8.50, 5.20, 30.00, 7.40, 0.50, 5.20, 12.00, 4.80, 
                    30.00, 3.50, 7.40, 0.50, 12.00, 5.20, 9.00, 4.80, 30.00, 8.50, 9.00],
    'quantidade': [1, 2, 2, 1, 1, 2, 10, 1, 1, 2, 1, 3, 1, 20, 2, 4, 1, 1, 2, 3, 1],
    'data': ['2023-10-01', '2023-10-01', '2023-10-01', '2023-10-02', '2023-10-02', '2023-10-03', '2023-10-03', '2023-10-04', '2023-10-04', '2023-10-05',
             '2023-10-05', '2023-10-06', '2023-10-06', '2023-10-07', '2023-10-07', '2023-10-08', '2023-10-08', '2023-10-09', '2023-10-09', '2023-10-10', '2023-10-10'],
    'custo_unidade': [22.00, 5.00, 5.00, 3.80, 22.00, 5.50, 0.20, 3.80, 8.00, 3.20, 22.00, 2.10, 5.50, 0.20, 8.00, 3.80, 6.50, 3.20, 22.00, 5.00, 6.50]
}

df = pd.DataFrame(data)
df = df.drop_duplicates()

df['data'] = pd.to_datetime(df['data']).dt.strftime('%d/%m/%Y')

df['total_venda'] = df['valor_venda'] * df['quantidade']
df['lucro'] = df['total_venda'] - (df['custo_unidade'] * df['quantidade'])

sns.set_theme(style="whitegrid")
plt.rcParams['font.sans-serif'] = 'Arial'
plt.rcParams['font.family'] = 'sans-serif'

def finalizar_grafico(titulo, nome_arquivo):
    plt.title(titulo, fontsize=14, fontweight='bold', pad=25, color='#2c3e50')
    plt.tight_layout()
    plt.savefig(nome_arquivo, dpi=300)
    plt.show()

plt.figure(figsize=(16, 8))
df_g1 = df.groupby(['data', 'produto'])['total_venda'].sum().reset_index()
df_g1['data_dt'] = pd.to_datetime(df_g1['data'], format='%d/%m/%Y')
df_g1 = df_g1.sort_values('data_dt').drop(columns=['data_dt'])

g1 = sns.barplot(data=df_g1, x='data', y='total_venda', hue='produto', palette='tab10')
for container in g1.containers:
    labels = [f'R$ {v:.2f}' if v > 0 else '' for v in container.datavalues]
    g1.bar_label(container, labels=labels, padding=6, fontsize=8.5, fontweight='bold', rotation=45)

for i in range(len(df_g1['data'].unique()) - 1):
    plt.axvline(x=i + 0.5, color='#b2bec3', linestyle='--', linewidth=1.2, alpha=0.8)

plt.xticks(rotation=45, ha='right', fontsize=10)
plt.ylabel('Valor Vendido (R$)')
plt.xlabel('Data da Venda (Padrão BR)')
plt.ylim(0, df_g1['total_venda'].max() * 1.3)
plt.legend(title='Produtos', bbox_to_anchor=(1.01, 1), loc='upper left')
finalizar_grafico('1. Detalhamento Diário: O que foi comprado e o valor exato ganho', '01_faturamento_diario.png')


plt.figure(figsize=(13, 8))
df_g2 = df.copy()
df_g2['data_dt'] = pd.to_datetime(df_g2['data'], format='%d/%m/%Y')
df_g2 = df_g2.sort_values(by=['data_dt', 'produto']).reset_index(drop=True)

plt.hlines(y=df_g2.index, xmin=0, xmax=df_g2['total_venda'], color='lightgray', alpha=0.7, linewidth=2)
g2 = sns.scatterplot(data=df_g2, x='total_venda', y=df_g2.index, hue='produto', s=180, palette='Dark2')

for idx, row in df_g2.iterrows():
    info_venda = f"  [{row['data']}] {row['produto']} ➔ R$ {row['total_venda']:.2f} ({row['quantidade']} un)"
    plt.text(row['total_venda'], idx, info_venda, va='center', fontsize=9.5, fontweight='bold')

plt.yticks([])
plt.ylabel('Linha do Tempo das Vendas (Antigas para Recentes)', labelpad=10)
plt.xlabel('Valor Total da Venda (R$)')
plt.xlim(0, df_g2['total_venda'].max() * 1.65)
plt.legend(title='Produto', bbox_to_anchor=(1.01, 1), loc='upper left')
finalizar_grafico('2. Histórico Geral: Data, Produto, Quantidade e Valor', '02_linha_tempo.png')


plt.figure(figsize=(9, 9))
df_g3 = df.groupby('produto')['lucro'].sum().sort_values(ascending=False)
labels_g3 = [f'{prod}\nLucro: R$ {val:.2f}' for prod, val in zip(df_g3.index, df_g3.values)]
plt.pie(df_g3, labels=labels_g3, autopct='%1.1f%%', startangle=130, 
        colors=sns.color_palette('pastel', len(df_g3)), pctdistance=0.75,
        wedgeprops={'edgecolor': 'white', 'linewidth': 1.5})
finalizar_grafico('3. Divisão de Lucro Real por Nome de Produto', '03_lucro_pizza.png')


plt.figure(figsize=(15, 6))
df_g4 = df.groupby('data')['total_venda'].sum().reset_index()
df_g4['data_dt'] = pd.to_datetime(df_g4['data'], format='%d/%m/%Y')
df_g4 = df_g4.sort_values('data_dt')

g4 = sns.lineplot(data=df_g4, x='data', y='total_venda', marker='o', markersize=10, color='#2c3e50', linewidth=3)
for idx, row in df_g4.iterrows():
    prods_dia = df[df['data'] == row['data']]['produto'].unique()
    lista_prods = ", ".join(prods_dia)
    texto_linha = f"R$ {row['total_venda']:.2f}\n[{lista_prods}]"
    plt.text(row['data'], row['total_venda'] + 2, texto_linha, ha='center', va='bottom', fontsize=8, fontweight='bold',
             bbox=dict(boxstyle='round,pad=0.2', edgecolor='orange', facecolor='lightyellow', alpha=0.8))

plt.xticks(rotation=45, ha='right')
plt.ylabel('Faturamento Total do Dia (R$)')
plt.xlabel('Data')
plt.ylim(0, df_g4['total_venda'].max() * 1.3)
finalizar_grafico('4. Linha de Evolução Diária do Caixa com os Produtos Vendidos', '04_evolucao_detalhada.png')


df['data_dt'] = pd.to_datetime(df['data'], format='%d/%m/%Y')
df_g5_ordenado = df.sort_values('data_dt')

grade = sns.relplot(
    data=df_g5_ordenado,
    x='total_venda',
    y='lucro',
    hue='produto',
    col='data',       
    col_wrap=5,       
    kind='scatter',
    s=180,            
    edgecolor='black',
    palette='Set1',
    alpha=0.9
)

grade.set_titles("{col_name}", weight='bold', color='#2c3e50')
grade.set_axis_labels("Valor da Venda (R$)", "Lucro Gerado (R$)")
plt.subplots_adjust(top=0.85)
grade.fig.suptitle('5. Análise de Venda: Relação de Preço e Lucro Separada por Dia', fontsize=15, fontweight='bold', color='#2c3e50')
grade.savefig('05_dispersao_detalhado.png', dpi=300)
plt.show()


plt.figure(figsize=(13, 7))
df_g6 = df.sort_values(by='quantidade', ascending=False).reset_index(drop=True)
g6 = sns.barplot(data=df_g6, x='quantidade', y=df_g6.index.astype(str), hue='produto', palette='tab20', dodge=False)

for idx, row in df_g6.iterrows():
    texto_qtd = f" Levou {row['quantidade']} unidades em {row['data']} (Total: R$ {row['total_venda']:.2f})"
    plt.text(row['quantidade'], idx, texto_qtd, va='center', fontsize=9, fontweight='bold', color='black')

plt.yticks([])
plt.ylabel('Registros de Compras no Caixa', labelpad=10)
plt.xlabel('Quantidade de Itens Comprados')
plt.xlim(0, df_g6['quantidade'].max() * 1.55)
plt.legend(title='Produto Comprado', bbox_to_anchor=(1.01, 1), loc='upper left')
finalizar_grafico('6. Rastreamento de Volume: Quantidades levadas por Produto e Data', '06_quantidade_real.png')

print("📊 Pronto! Gráficos salvos com o mais alto padrão de organização.")
