# cod.dash
import dash
from dash import dcc, html
import pandas as pd
import plotly.express as px
import plotly.figure_factory as ff
import numpy as np

# 1. Carregamento e Preparação dos Dados
# Alterado para o nome do arquivo que você possui no ambiente
df = pd.read_csv('ecommerce_estatistica.csv')

# 2. Criação dos Gráficos Interativos (Plotly)

# 1. Histograma - Preços
fig_hist = px.histogram(df, x='Preço', nbins=30, marginal='box', 
                        title='Histograma: Distribuição de Preços',
                        color_discrete_sequence=['skyblue'])

# 2. Dispersão - Preço vs Nota
fig_scatter = px.scatter(df, x='Preço', y='Nota', opacity=0.6,
                         title='Dispersão: Preço vs Nota',
                         color_discrete_sequence=['coral'])

# 3. Mapa de Calor - Correlação
numeric_cols = ['Nota', 'N_Avaliações', 'Desconto', 'Preço', 'Qtd_Vendidos_Cod']
corr = df[numeric_cols].corr()
fig_heatmap = px.imshow(corr, text_auto=".2f", aspect="auto",
                        color_continuous_scale='RdBu_r', center=0,
                        title='Mapa de Calor: Correlação entre Variáveis')

# 4. Gráfico de Barra - Preço Médio por Gênero
avg_price_gender = df.groupby('Gênero')['Preço'].mean().sort_values(ascending=False).reset_index()
fig_bar = px.bar(avg_price_gender, x='Preço', y='Gênero', orientation='h',
                 title='Gráfico de Barra: Preço Médio por Gênero',
                 color='Preço', color_continuous_scale='magma')

# 5. Gráfico de Pizza - Distribuição por Gênero
gender_counts = df['Gênero'].value_counts().reset_index()
gender_counts.columns = ['Gênero', 'Contagem']
fig_pie = px.pie(gender_counts, values='Contagem', names='Gênero',
                 title='Gráfico de Pizza: Distribuição de Produtos por Gênero',
                 color_discrete_sequence=px.colors.qualitative.Set3)

# 6. Gráfico de Densidade - Notas
# O Plotly usa o figure_factory para KDE (Kernel Density Estimation)
fig_density = ff.create_distplot([df['Nota'].dropna()], ['Nota'], 
                                 show_hist=False, colors=['purple'])
fig_density.update_layout(title='Gráfico de Densidade: Distribuição das Notas')

# 7. Gráfico de Regressão - Avaliações vs Vendas
fig_reg = px.scatter(df, x='N_Avaliações', y='Qtd_Vendidos_Cod', trendline='ols',
                     title='Gráfico de Regressão: Avaliações vs Vendas',
                     render_mode='svg')
fig_reg.update_traces(marker=dict(opacity=0.3), selector=dict(mode='markers'))

# 3. Configuração do Layout do Dash
app = dash.Dash(__name__)

app.layout = html.Div(style={'fontFamily': 'Arial', 'padding': '20px', 'backgroundColor': '#f4f4f4'}, children=[
    html.H1("Dashboard de Estatísticas E-commerce", style={'textAlign': 'center', 'color': '#333'}),
    
    # Primeira Linha
    html.Div([
        html.Div([dcc.Graph(figure=fig_hist)], style={'width': '49%', 'display': 'inline-block'}),
        html.Div([dcc.Graph(figure=fig_scatter)], style={'width': '49%', 'display': 'inline-block'}),
    ], style={'display': 'flex', 'justifyContent': 'space-between'}),

    # Segunda Linha
    html.Div([
        html.Div([dcc.Graph(figure=fig_heatmap)], style={'width': '49%', 'display': 'inline-block'}),
        html.Div([dcc.Graph(figure=fig_bar)], style={'width': '49%', 'display': 'inline-block'}),
    ], style={'display': 'flex', 'justifyContent': 'space-between', 'marginTop': '20px'}),

    # Terceira Linha
    html.Div([
        html.Div([dcc.Graph(figure=fig_pie)], style={'width': '49%', 'display': 'inline-block'}),
        html.Div([dcc.Graph(figure=fig_density)], style={'width': '49%', 'display': 'inline-block'}),
    ], style={'display': 'flex', 'justifyContent': 'space-between', 'marginTop': '20px'}),

    # Quarta Linha (Largura Total)
    html.Div([
        dcc.Graph(figure=fig_reg)
    ], style={'marginTop': '20px', 'backgroundColor': 'white', 'padding': '10px'})
])

# 4. Execução do Servidor
if __name__ == '__main__':
    app.run_server(debug=True)
