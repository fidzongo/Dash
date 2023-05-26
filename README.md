# Dash
Dashboard avec Dash

# Installation
```python
pip install dash pandas matplotlib numpy
ou
pip install -r requirements.txt
```

# Code de l'application

```python
import dash
import pandas as pd
#import matplotlib.pyplot as plt
#import plotly.express as px
import plotly.graph_objects as go
#import plotly.subplots as sp

from dash import dcc
from dash import html
from dash import dash_table
from dash.dependencies import Output,Input
from plotly.subplots import make_subplots

#Chargement du dataframe
df = pd.read_csv('nba_2013.csv', sep=';', header=0)
df_1 = df[(df.bref_team_id != "TOP") & (df.pos != "G")]
df_slider = df_1.pos.unique()

#Mise en page
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']
app = dash.Dash(__name__, external_stylesheets=external_stylesheets,suppress_callback_exceptions=True)

#Layoute
app.layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id = 'page-content')
])

#Page principale
index_page = html.Div([
    html.H1('Evaluation Module Dash', style={'color' : 'aquamarine', 'textAlign': 'center'}),
    html.Button(dcc.Link('Comparaison des joeurs', href='/players-comparison')),
    html.Br(),
    html.Br(),
    html.Button(dcc.Link('Comparaison des équipes', href='/teams-comparison'))
], style={'alignItems': 'center', 'textAlign': 'center'})

#Page 1 - comparaison des équipes
layout_1 = html.Div([
    html.Div(className='row', children=[
        html.Div(children=[
                html.Label(['Rookie:'], style={'font-weight': 'bold', "text-align": "center"}),
                dcc.Dropdown(
                    id='dropdown-rookie',
		            options= [{'label': i, 'value': i} for i in df.age.loc[df.age < 24].unique()],
                    value = df['age'].min()
                )
            ], style=dict(width='50%')),
        html.Div(children=[
                html.Label(['Senior:'], style={'font-weight': 'bold', "text-align": "center"}),
                dcc.Dropdown(
                    id='dropdown-senior',
		            options= [{'label': i, 'value': i} for i in df.age.loc[df.age > 24].unique()],
                    value = df['age'].max()
                )
            ], style=dict(width='50%')),
    ],style=dict(display='flex')),

    #Rookie Players
    html.H6('Rookie (Joueurs de moins de 24 ans):'),

    #Table affichant les statistiques des joeurs de moins de 24 ans
    html.Div(id = 'table-rookie-player'),

    #Senior Players
    html.H6('Senior (Joeurs de plus de 24 ans):'),

    #Table affichant les statistiques des joeurs de plus de 24 ans
    html.Div(id = 'table-senior-player'),

    #Saut de ligne
    html.Br(),

    #Retour arrière vers la page de garde
    html.Div(
        html.Button(dcc.Link('Revenir à la page de garde', href='/')),
        style={'textAlign': 'center'}
    )
 ]
)

#Callback Rookie players
@app.callback(Output(component_id='table-rookie-player', component_property='children'),
            [Input(component_id='dropdown-rookie', component_property='value')])
def update_table_rookie(filter_rookie):
    if filter_rookie != None:
       df_2 = df_1[df_1["age"] == filter_rookie]
    else:
       df_2 = pd.DataFrame()
    #Création de la table
    table = dash_table.DataTable(df_2.to_dict('records'), [{"name": i, "id": i} for i in df_2.columns])
    return table

#Callback Senior players
@app.callback(Output(component_id='table-senior-player', component_property='children'),
            [Input(component_id='dropdown-senior', component_property='value')])                        
def update_table_senior(filter_senior):
    if filter_senior != None:                                                                                  
       df_2 = df_1[df_1["age"] == filter_senior]
    else:
       df_2 = pd.DataFrame()
    #Création de la table
    table = dash_table.DataTable(df_2.to_dict('records'), [{"name": i, "id": i} for i in df_2.columns])
    return table

#Page 2 - comparaison des équipes
layout_2 = html.Div([
    html.Div(className='row', children=[
        html.Div(children=[
                html.Label(['Choix 1 (Caractéristiques):'], style={'font-weight': 'bold', "text-align": "center"}),
                dcc.Dropdown(
                    id='dropdown-choice-1',
                    options= [{'label': i, 'value': i} for i in df.loc[:, ~df.columns.isin(['player', 'pos', 'bref_team_id', 'season'])]],
                    value = 'age'
                )
            ], style=dict(width='50%')),
        html.Div(children=[
                html.Label(['Choix 2 (Caractéristiques - Variation en fonction de la position (pos)):'], style={'font-weight': 'bold', "text-align": "center"}),
                dcc.Dropdown(
                    id='dropdown-choice-2',
                    options= [{'label': i, 'value': i} for i in df.loc[:, ~df.columns.isin(['player', 'pos', 'bref_team_id', 'season'])]],
                    value = 'age'
                )
            ], style=dict(width='50%')),
    ],style=dict(display='flex')),

    #Add dynamic plot
    dcc.Graph(id="plot-1"),
    
    #Slider pour varier les graphiques
    html.Label(['Pour varier le choix 2:'], style={'font-weight': 'bold', "text-align": "center"}),
    dcc.Slider(id = 'slider_1',
                        min = 0,
                        max = len(df_slider)-1,
                        marks={str(index): str(df_slider[index]) for index in range(len(df_slider))},
                        ),
    
    #Retour arrière vers la page de garde
    html.Div(
        html.Button(dcc.Link('Revenir à la page de garde', href='/')),
        style={'textAlign': 'center'}
    )

 ]
)
#Choice 1 & Choice 2
@app.callback(Output(component_id='plot-1', component_property='figure'),
            [Input(component_id='dropdown-choice-1', component_property='value'),
             Input(component_id='dropdown-choice-2', component_property='value'),
             Input(component_id='slider_1', component_property='value')])
def update_plot_1(filter_choice_1, filter_choice_2, filter_slider_1):
    fig = make_subplots(rows=1, cols=2, x_title='Top 5 teams')
    fig.update_layout(title_text='Comparaison des équipes', legend_title_text='Choix')

    #Choice 1
    if filter_choice_1 != None :                                                                                  
        df_2 = df_1.groupby(by=['bref_team_id'])[filter_choice_1].sum()
        df_3 = df_2.nlargest(5)
        fig.add_trace(
            go.Bar(name="Choix 1 = {}".format(filter_choice_1), x=df_3.index, y=df_3.values),
            row=1, 
            col=1
        )
    else:
        df_3 = pd.DataFrame()
        fig.add_trace(
            go.Bar(name="Choix 1 = {}".format(filter_choice_1), x=df_3.index, y=df_3.values),
            row=1, 
            col=1
        )  
    #Choice 2
    #{'0': 'SF', '1': 'C', '2': 'PF', '3': 'SG', '4': 'PG', '5': 'F'}
    if (filter_slider_1 == 0 or filter_slider_1 == None):
        col_value = 'SF'
    elif filter_slider_1 == 1: 
        col_value = 'C'
    elif filter_slider_1 == 2:
        col_value = 'PF'
    elif filter_slider_1 == 3:
        col_value = 'SG'
    elif filter_slider_1 == 4:
        col_value = 'PG'
    else:
        col_value = 'F'
 
    if filter_choice_2 != None : 
        df_choice_2 = df_1[df_1['pos'] == col_value]                                                                              
        df_2 = df_choice_2.groupby(by=['bref_team_id'])[filter_choice_2].sum()
        df_3 = df_2.nlargest(5)
        fig.add_trace(
            go.Bar(name="Choix 1 = {}".format(filter_choice_2), x=df_3.index, y=df_3.values),
            row=1, 
            col=2
        )  
    else:
        df_3 = pd.DataFrame()
        fig.add_trace(
            go.Bar(name="Choix 1 = {}".format(filter_choice_2), x=df_3.index, y=df_3.values),
            row=1, 
            col=2
        )  

    return fig
    
#Index update
@app.callback(dash.dependencies.Output('page-content', 'children'),
              [dash.dependencies.Input('url', 'pathname')])
def display_page(pathname):
    if pathname == '/players-comparison':                                                                                   
        return layout_1
    elif pathname == '/teams-comparison':
        return layout_2
    else:
        return index_page

if __name__ == '__main__':
    app.run_server(debug=True, port=8080)
```
