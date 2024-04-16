# Phonepe_pulse_project
import streamlit as st
from streamlit_option_menu import option_menu
import pandas as pd
import psycopg2
import plotly.express as px
import requests
import json
from streamlit_option_menu import option_menu
from PIL import Image

#datafreame creation
#sql connection
mydb= psycopg2.connect(host="localhost",
                        user="postgres",
                        port=5432,
                        database="phonepe_data",
                        password="guvi")
cursor=mydb.cursor()

#aggre_transaction_df
cursor.execute("SELECT * FROM aggregated_transaction")
mydb.commit()
table1= cursor.fetchall()

Aggre_transaction = pd.DataFrame(table1,columns=("States","Years","Quarter","Transaction_type","Transaction_count",
                                                 "Transaction_amount"))


#aggre_user_df
cursor.execute("SELECT * FROM aggregated_user")
mydb.commit()
table2= cursor.fetchall()

Aggre_user = pd.DataFrame(table2,columns=("States","Years","Quarter","Brands","Transaction_count",
                                                 "Percentage"))


#map_transaction_df
cursor.execute("SELECT * FROM map_transaction")
mydb.commit()
table3= cursor.fetchall()

Map_transaction= pd.DataFrame(table3,columns=("States","Years","Quarter","Districts","Transaction_count",
                                                 "Transaction_amount"))


#map_user_df
cursor.execute("SELECT * FROM map_user")
mydb.commit()
table4= cursor.fetchall()

Map_user= pd.DataFrame(table4,columns=("States","Years","Quarter","Districts","RegisteredUsers",
                                                 "AppOpens"))

#top_transaction_df
cursor.execute("SELECT * FROM top_transaction")
mydb.commit()
table5= cursor.fetchall()

Top_transaction= pd.DataFrame(table5,columns=("States","Years","Quarter","Pincodes","Transaction_count",
                                                 "Transaction_amount"))

#top_user_df
cursor.execute("SELECT * FROM top_user")
mydb.commit()
table6= cursor.fetchall()

Top_user= pd.DataFrame(table6,columns=("States","Years","Quarter","Pincodes","RegisteredUsers"))


def Transaction_amount_count_Y(df, year):

    tacy=df[df["Years"]==year]
    tacy.reset_index(drop = True,inplace=True)

    tacyg=tacy.groupby("States")[["Transaction_count","Transaction_amount"]].sum()
    tacyg.reset_index(inplace=True)

    col1,col2 = st.columns(2)
    with col1:
        fig_amount= px.bar(tacyg,x="States",y="Transaction_amount",title=f"{year} Transaction Amount",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl,height=650,width=600)
        st.plotly_chart(fig_amount)
    with col2:
        fig_count= px.bar(tacyg,x="States",y="Transaction_count",title=f"{year} Transaction Count",
                            color_discrete_sequence=px.colors.sequential.Bluered_r,height=650,width=600)
        st.plotly_chart(fig_count)

    col1,col2 = st.columns(2)
    with col1:
    
        url="https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
        response=requests.get(url)
        data1= json.loads(response.content)
        state_name=[]
        for feature in data1["features"]:
            feature=feature["properties"]["ST_NM"].replace("&","and")
            state_name.append(feature)

        state_name.sort()

        fig_india_1= px.choropleth(tacyg,geojson=data1, locations= "States", featureidkey= "properties.ST_NM",
                                    color="Transaction_amount",color_continuous_scale="Rainbow",
                                    range_color=(tacyg["Transaction_amount"].min(), tacyg["Transaction_amount"].max()),
                                    hover_name="States", title= f"{year}Transaction Amount",fitbounds="locations",
                                    height=600,width=600)
        
        fig_india_1.update_geos(visible=False)
        st.plotly_chart(fig_india_1)

    with col2:
        fig_india_2= px.choropleth(tacyg,geojson=data1, locations= "States", featureidkey= "properties.ST_NM",
                                color="Transaction_count",color_continuous_scale="Rainbow",
                                range_color=(tacyg["Transaction_count"].min(), tacyg["Transaction_count"].max()),
                                hover_name="States", title= f"{year}Transaction Count",fitbounds="locations",
                                height=600,width=600)

        fig_india_2.update_geos(visible=False)
        st.plotly_chart(fig_india_2)

    return tacy



def Transaction_amount_count_Y_Q(df, quarter):
    tacy=df[df["Quarter"]==quarter]
    tacy.reset_index(drop = True,inplace=True)

    tacyg=tacy.groupby("States")[["Transaction_count","Transaction_amount"]].sum()
    tacyg.reset_index(inplace=True)

    col1,col2 = st.columns(2)

    with col1:
        fig_amount= px.bar(tacyg,x="States",y="Transaction_amount",title=f"{tacy['Years'].min()} YEAR {quarter} Quarter Transaction Amount",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl,height=650,width=600)
        st.plotly_chart(fig_amount)

    with col2:

        fig_count= px.bar(tacyg,x="States",y="Transaction_count",title=f"{tacy['Years'].min()} YEAR {quarter} Quarter Transaction Count",
                            color_discrete_sequence=px.colors.sequential.Bluered_r,height=650,width=600)
        st.plotly_chart(fig_count)

    col1,col2 = st.columns(2)

    with col1:

        url="https://gist.githubusercontent.com/jbrobst/56c13bbbf9d97d187fea01ca62ea5112/raw/e388c4cae20aa53cb5090210a42ebb9b765c0a36/india_states.geojson"
        response=requests.get(url)
        data1= json.loads(response.content)
        state_name=[]
        for feature in data1["features"]:
            feature=feature["properties"]["ST_NM"].replace("&","and")
            state_name.append(feature)

        state_name.sort()

        fig_india_1= px.choropleth(tacyg,geojson=data1, locations= "States", featureidkey= "properties.ST_NM",
                                    color="Transaction_amount",color_continuous_scale="Rainbow",
                                    range_color=(tacyg["Transaction_amount"].min(), tacyg["Transaction_amount"].max()),
                                    hover_name="States", title= f"{tacy['Years'].min()} YEAR {quarter} Quarter Transaction Amount",fitbounds="locations",
                                    height=600,width=600)
        
        fig_india_1.update_geos(visible=False)
        st.plotly_chart(fig_india_1)

    with col2:
        fig_india_2= px.choropleth(tacyg,geojson=data1, locations= "States", featureidkey= "properties.ST_NM",
                                color="Transaction_count",color_continuous_scale="Rainbow",
                                range_color=(tacyg["Transaction_count"].min(), tacyg["Transaction_count"].max()),
                                hover_name="States", title= f"{tacy['Years'].min()} YEAR {quarter} Quarter Transaction Count",fitbounds="locations",
                                height=600,width=600)

        fig_india_2.update_geos(visible=False)
        st.plotly_chart(fig_india_2)



def Aggre_Trans_Transaction_Type(df, state):

    tacy=df[df["States"]== state]
    tacy.reset_index(drop = True,inplace=True)

    tacyg=tacy.groupby("Transaction_type")[["Transaction_count","Transaction_amount"]].sum()
    tacyg.reset_index(inplace=True)
    col1,col2 = st.columns(2)
    with col1:
        fig_pie_1= px.pie(data_frame= tacyg, names= "Transaction_type", values="Transaction_amount",
                        width=600, title= f"{state} Transaction Amount Type",hole=0.3)


        st.plotly_chart(fig_pie_1)


    with col2:
        fig_pie_2= px.pie(data_frame= tacyg, names= "Transaction_type", values="Transaction_count",
                        width=600, title= f"{state} Transaction Count Type",hole=0.3)


        st.plotly_chart(fig_pie_2)


#Aggre_user_plot_1
def Aggre_user_plot_1(df, year):
    aguy= Aggre_user[Aggre_user["Years"]==year]
    aguy.reset_index(drop=True,inplace= True)

    aguyg= pd.DataFrame(aguy.groupby("Brands")["Transaction_count"].sum())
    aguyg.reset_index(inplace= True)


    fig_bar_1= px.bar(aguyg,x="Brands",y="Transaction_count", title= f"{year} Brands and Transaction Count",
                    width=1000,color_discrete_sequence=px.colors.sequential.Sunsetdark_r,hover_name="Brands")
    st.plotly_chart(fig_bar_1)

    return aguy


#Aggre_user_plot_2
def Aggre_user_plot_2(df, quarter):
    aguyq= df[df["Quarter"]==quarter]
    aguyq.reset_index(drop=True,inplace= True)

    aguyqg= pd.DataFrame(aguyq.groupby("Brands")["Transaction_count"].sum())
    aguyqg.reset_index(inplace= True)

    fig_bar_1= px.bar(aguyqg,x="Brands",y="Transaction_count", title=f"{quarter} Brands and Transaction Count",
                    width=1000,color_discrete_sequence=px.colors.sequential.Agsunset_r,hover_name="Brands")
    st.plotly_chart(fig_bar_1)

    return aguyq


#Aggre_user_plot_3
def Aggre_user_plot_3(df, state):
    auyqs= df[df["States"] == state]
    auyqs.reset_index(drop=True, inplace=True)

    fig_scat_1= px.scatter(auyqs,x="Brands",y="Transaction_count",hover_data= "Percentage",
                        title="Brands,Transaction count,Percentage",width=1000)

    st.plotly_chart(fig_scat_1)

    return auyqs


#Map_Trans_districts
def Map_Trans_District(df, state):

    tacy=df[df["States"]== state]
    tacy.reset_index(drop = True,inplace=True)

    tacyg=tacy.groupby("Districts")[["Transaction_count","Transaction_amount"]].sum()
    tacyg.reset_index(inplace=True)

    col1,col2 = st.columns(2)
    with col1:

        fig_bar_1= px.bar(tacyg, y= "Districts", x ="Transaction_count",orientation= "h",height=600,
                        width=600, title= f"{state} District Transaction Amount Type",color_discrete_sequence=px.colors.sequential.Aggrnyl_r)


        st.plotly_chart(fig_bar_1)

    with col2:
        fig_bar_2= px.bar(tacyg, y= "Districts", x ="Transaction_count",orientation= "h",height=600,
                        width=600, title= f"{state} District Transaction Count Type",color_discrete_sequence=px.colors.sequential.Jet)


        st.plotly_chart(fig_bar_1)


#Map_user_plot_1

def Map_user_plot_1(df,year):
    muy= df[df["Years"]==year]
    muy.reset_index(drop=True,inplace= True)

    muyg= pd.DataFrame(muy.groupby("States")[["RegisteredUsers", "AppOpens"]].sum())
    muyg.reset_index(inplace= True)


    fig_line_1= px.line(muyg,x="States",y=["RegisteredUsers","AppOpens"],
                        title=f"{year},RegisteredUsers,AppOpens",width=1000,height=800,markers=True)

    st.plotly_chart(fig_line_1)

    return muy


#Map_user_plot_2
def Map_user_plot_2(df,quarter):
    muyq= df[df["Quarter"]==quarter]
    muyq.reset_index(drop=True,inplace= True)

    muyqg= pd.DataFrame(muyq.groupby("States")[["RegisteredUsers", "AppOpens"]].sum())
    muyqg.reset_index(inplace= True)


    fig_line_1= px.line(muyqg,x="States",y=["RegisteredUsers","AppOpens"],
                        title=f"{df['Years'].min()} Year {quarter} Quarter RegisteredUsers,AppOpens",width=1000,height=800,markers=True)

    st.plotly_chart(fig_line_1)

    return muyq


#Map_user_plot_3
def Map_user_plot_3(df,states):
    muyqs= df[df["States"]==states]
    muyqs.reset_index(drop=True,inplace= True)
    col1,col2=st.columns(2)
    with col1:
        fig_map_user_line_1 = px.bar(muyqs, x="RegisteredUsers",y="Districts",
                                    title=f"{states} Registered Users",height=800)
        
        st.plotly_chart(fig_map_user_line_1)
    with col2:
        fig_map_user_line_2= px.bar(muyqs, x="AppOpens",y="Districts",
                                    title=f"{states} AppOpens",height=800,color_discrete_sequence=px.colors.sequential.Jet)
        
        st.plotly_chart(fig_map_user_line_2)


#Top_trans_plot_1
def Top_trans_plot_1(df,state):
    tty= df[df["States"]==state]
    tty.reset_index(drop=True,inplace= True)
     
    col1,col2=st.columns(2)
    with col1:
        fig_top_trans_bar_1= px.bar(tty, y= "Transaction_amount", x ="Quarter",hover_data="Pincodes",
                        width=600,height=800, title="Transaction Amount",color_discrete_sequence=px.colors.sequential.Aggrnyl_r)


        st.plotly_chart(fig_top_trans_bar_1)

    with col2:
        fig_top_trans_bar_2= px.bar(tty, y= "Transaction_count", x ="Quarter",hover_data="Pincodes",
                        width=600,height=800 ,title="Transaction Count",color_discrete_sequence=px.colors.sequential.Jet)

        st.plotly_chart(fig_top_trans_bar_2)


#Top_user_plot_1
def Top_user_plot_1(df, year):
    tuy= df[df["Years"]==year]
    tuy.reset_index(drop=True,inplace= True)

    tuyg= pd.DataFrame(tuy.groupby(["States","Quarter"])["RegisteredUsers"].sum())
    tuyg.reset_index(inplace= True)

    fig_top_plot_1= px.bar(tuyg,x="States",y="RegisteredUsers",color="Quarter", width=1000, height=800,
                            color_discrete_sequence=px.colors.sequential.Blackbody,hover_name="States",
                            title=f"{year}Registered User")
    st.plotly_chart(fig_top_plot_1)

    return tuy


#Top_user_plot_2
def Top_user_plot_2(df, state):
    tuys= df[df["States"]==state]
    tuys.reset_index(drop=True,inplace= True)

    fiq_top_plot_2=px.bar(tuys,x="Quarter",y="RegisteredUsers",width=1000,height=800,
                        title="Registered Users,Pincodes, Quarter",color="RegisteredUsers",hover_data="Pincodes",
                        color_continuous_scale=px.colors.sequential.Magenta)

    st.plotly_chart(fiq_top_plot_2)

#Top_charts
def top_chart_transaction_amount(table_name):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select states, sum(transaction_amount)as transaction_amount
                from {table_name}
                group by states
                order by transaction_amount desc
                limit(10);'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()


        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("states","transaction_amount"))
            fig_df_1=px.bar(df_1,x="states",y="transaction_amount",
                            title= "Top 10 0f Transaction Amount",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select states, sum(transaction_amount)as transaction_amount
                from {table_name}
                group by states
                order by transaction_amount
                limit(10);'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("states","transaction_amount"))
            fig_df_2=px.bar(df_2,x="states",y="transaction_amount",
                            title= "Least 10 of Transaction Amount",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select states, avg(transaction_amount) as transaction_amount
                from {table_name}
                group by states
                order by transaction_amount asc;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("states","transaction_amount"))
        fig_df_3=px.bar(df_3,x="transaction_amount",y="states",orientation="h",
                        title= "Average of Transaction Amount",height=800,width=1000,hover_name="states",
                        color_discrete_sequence=px.colors.sequential.Viridis_r)
        st.plotly_chart(fig_df_3)


#Top_chart_count
def top_chart_transaction_count(table_name):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select states, sum(transaction_count)as transaction_count
                from {table_name}
                group by states
                order by transaction_count desc
                limit(10);'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()

        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("states","transaction_count"))
            fig_df_1=px.bar(df_1,x="states",y="transaction_count",
                            title= "Top 10 0f Transaction Count",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select states, sum(transaction_count)as transaction_count
                from {table_name}
                group by states
                order by transaction_count
                limit(10);'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("states","transaction_count"))
            fig_df_2=px.bar(df_2,x="states",y="transaction_count",
                            title= "Least 10 of Transaction Count",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select states, avg(transaction_count) as transaction_count
                from {table_name}
                group by states
                order by transaction_count asc;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("states","transaction_count"))
        fig_df_3=px.bar(df_3,x="transaction_count",y="states",orientation="h",
                        title= "Average of Transaction Count",height=800,width=1000,hover_name="states",
                        color_discrete_sequence=px.colors.sequential.Viridis)
        st.plotly_chart(fig_df_3)

#Top chart registered user

def top_chart_registered_users(table_name,state):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()

        #plot_1
        query1=f'''select districts,sum(registeredusers) as registeredusers 
                from {table_name}
                where states='{state}'
                group by districts
                order by registeredusers desc
                limit(10);'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()
        
        
        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("districts","registeredusers"))
            fig_df_1=px.bar(df_1,x="districts",y="registeredusers",
                            title= "Top 10 of Registeredusers",height=650,width=600,hover_name="districts",
                            color_discrete_sequence=px.colors.sequential.gray)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select districts,sum(registeredusers) as registeredusers 
                from {table_name}
                where states='{state}'
                group by districts
                order by registeredusers
                limit(10);'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("districts","registeredusers"))
            fig_df_2=px.bar(df_2,x="districts",y="registeredusers",
                            title= "Least 10 of Registeredusers",height=650,width=600,hover_name="districts",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select districts,avg(registeredusers) as registeredusers 
                        from {table_name}
                        where states='{state}'
                        group by districts
                        order by registeredusers asc;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("districts","registeredusers"))
        fig_df_3=px.bar(df_3,x="registeredusers",y="districts",orientation="h",
                        title= "Average of Registeredusers",height=800,width=1000,hover_name="districts",
                        color_discrete_sequence=px.colors.sequential.Bluered)
        st.plotly_chart(fig_df_3)

#Top_chart_appopens
def top_chart_appopens(table_name,state):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select districts,sum(appopens) as appopens 
                from {table_name}
                where states='{state}'
                group by districts
                order by appopens desc
                limit(10);'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()

        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("districts","appopens"))
            fig_df_1=px.bar(df_1,x="districts",y="appopens",
                            title= "Top 10 of appopens",height=650,width=600,hover_name="districts",
                            color_discrete_sequence=px.colors.sequential.gray)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select districts,sum(appopens) as appopens 
                from {table_name}
                where states='{state}'
                group by districts
                order by appopens
                limit(10);'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("districts","appopens"))
            fig_df_2=px.bar(df_2,x="districts",y="appopens",
                            title= "Least 10 of appopens",height=650,width=600,hover_name="districts",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)

        #plot_3
        query3=f'''select districts,avg(appopens) as appopens 
                        from {table_name}
                        where states='{state}'
                        group by districts
                        order by appopens asc;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("districts","appopens"))
        fig_df_3=px.bar(df_3,x="appopens",y="districts",orientation="h",
                        title= "Average of appopens",height=800,width=1000,hover_name="districts",
                        color_discrete_sequence=px.colors.sequential.Bluered)
        st.plotly_chart(fig_df_3)


#Top_chart_registered_user
def top_chart_registered_user(table_name):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select states,sum(registeredusers) as registeredusers
                from {table_name}
                group by states
                order by registeredusers desc
                limit 10;'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()

        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("states","registeredusers"))
            fig_df_1=px.bar(df_1,x="states",y="registeredusers",
                            title= "Top 10 of Registeredusers",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.gray)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select states,sum(registeredusers) as registeredusers
                from {table_name}
                group by states
                order by registeredusers
                limit 10;'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("states","registeredusers"))
            fig_df_2=px.bar(df_2,x="states",y="registeredusers",
                            title= "Least 10 of Registeredusers",height=650,width=600,hover_name="states",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select states,avg(registeredusers) as registeredusers
                    from {table_name}
                    group by states
                    order by registeredusers;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("states","registeredusers"))
        fig_df_3=px.bar(df_3,x="registeredusers",y="states",orientation="h",
                        title= "Average of Registeredusers",height=800,width=1000,hover_name="states",
                        color_discrete_sequence=px.colors.sequential.Bluered)
        st.plotly_chart(fig_df_3)


#top_chart_transaction_type_count
def top_chart_transaction_type_count(table_name):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select transaction_type,sum(transaction_count) as transaction_count
                    from {table_name}
                    group by transaction_type
                    order by transaction_count;'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()

        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("transaction_type","transaction_count"))
            fig_df_1=px.bar(df_1,x="transaction_type",y="transaction_count",
                            title= "Transaction_type and Transaction Count Analysis",height=650,width=600,hover_name="transaction_type",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select transaction_type,sum(transaction_count) as transaction_count
                    from {table_name}
                    group by transaction_type
                    order by transaction_count desc;'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("transaction_type","transaction_count"))
            fig_df_2=px.bar(df_2,x="transaction_type",y="transaction_count",
                            title= "Transaction_type and Transaction Count Analysis",height=650,width=600,hover_name="transaction_type",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select transaction_type,avg(transaction_count) as transaction_count
                    from {table_name}
                    group by transaction_type
                    order by transaction_count;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("transaction_type","transaction_count"))
        fig_df_3=px.bar(df_3,x="transaction_count",y="transaction_type",orientation="h",
                        title= "Average of Transaction_type and Transaction Count Analysis",height=800,width=1000,hover_name="transaction_type",
                        color_discrete_sequence=px.colors.sequential.Bluered)
        st.plotly_chart(fig_df_3)

#top_chart_transaction_type_amount
def top_chart_transaction_type_amount(table_name):
        mydb= psycopg2.connect(host="localhost",
                                user="postgres",
                                port=5432,
                                database="phonepe_data",
                                password="guvi")
        cursor=mydb.cursor()
        #plot_1
        query1=f'''select transaction_type,sum(transaction_amount) as transaction_amount
                    from {table_name}
                    group by transaction_type
                    order by transaction_amount;'''


        cursor.execute(query1)
        table_1=cursor.fetchall()
        mydb.commit()

        col1,col2=st.columns(2)
        with col1:
            df_1=pd.DataFrame(table_1,columns=("transaction_type","transaction_amount"))
            fig_df_1=px.bar(df_1,x="transaction_type",y="transaction_amount",
                            title= "Transaction_type and Transaction Amount Analysis",height=650,width=600,hover_name="transaction_type",
                            color_discrete_sequence=px.colors.sequential.Aggrnyl)
            st.plotly_chart(fig_df_1)

        #plot_2
        query2=f'''select transaction_type,sum(transaction_amount) as transaction_amount
                    from {table_name}
                    group by transaction_type
                    order by transaction_amount desc;'''


        cursor.execute(query2)
        table_2=cursor.fetchall()
        mydb.commit()

        with col2:
            df_2=pd.DataFrame(table_2,columns=("transaction_type","transaction_amount"))
            fig_df_2=px.bar(df_2,x="transaction_type",y="transaction_amount",
                            title= "Transaction_type and Transaction Amount Analysis",height=650,width=600,hover_name="transaction_type",
                            color_discrete_sequence=px.colors.sequential.Agsunset_r)
            st.plotly_chart(fig_df_2)


        #plot_3
        query3=f'''select transaction_type,avg(transaction_amount) as transaction_amount
                    from {table_name}
                    group by transaction_type
                    order by transaction_amount;'''


        cursor.execute(query3)
        table_3=cursor.fetchall()
        mydb.commit()

        df_3=pd.DataFrame(table_3,columns=("transaction_type","transaction_amount"))
        fig_df_3=px.bar(df_3,x="transaction_amount",y="transaction_type",orientation="h",
                        title= "Average of Transaction_type and Transaction Amount Analysis",height=800,width=1000,hover_name="transaction_type",
                        color_discrete_sequence=px.colors.sequential.Bluered)
        st.plotly_chart(fig_df_3)

#Streamlit Part
st.set_page_config(layout="wide")
col1,col3=st.columns(2,gap="small")
with col1:

    st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/download.jpg",width=500)

with col3:

    st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/phonepepulse.png",width=500)


st.title(" :violet[Phonepe Pulse Data Visulalization and Data Exploration Project]")


with st.sidebar:
    st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/download.png")
    st.write("## :green[Phonepe pulse | Thigazhvan]")
    select=option_menu("Details",["About phonepe","About the Project","Data Visulalization","Top Charts"])

if select=="About phonepe":
    st.markdown("### [Explore about the PhonePe]|(https://www.phonepe.com/)")
    st.download_button("Download the App","https://www.phonepe.com/app-download/")
    st.video("C:/Users/MY COMPUTER/Desktop/Phone pe project/PhonePe Motion Graphics.mp4")
    st.markdown("## :violet[PhonePe]")
    st.write("### PhonePe is an Indian digital payments and financial services company headquartered in Bengaluru, Karnataka, India. PhonePe was founded in December 2015, by Sameer Nigam, Rahul Chari and Burzin Engineer. The PhonePe app, based on the Unified Payments Interface (UPI), went live in August 2016")
    st.write("### The PhonePe app is accessible in 11 Indian languages.[1] It enables users to perform various financial transactions such as sending and receiving money, recharging mobile and DTH services, topping up data cards, making utility payments, conducting in-store payments")
    st.markdown("## :violet[Payment systems]")
    st.write("### E-commerce payment system")
    st.write("### Mobile payment")
    st.write("### Digital wallet")
    st.write("### List of online payment service provider")
    st.write("### Payment gateway")
    st.write("### Fintech")
    st.write("### Financial services")
    st.write("### Mutual fund")
    st.write("### Insurance")
    st.write("### Digital gold")
    
elif select=="About the Project":
    #st.title(":blue[Data Visulalization and Data Exploration of Phonepe data]")
    st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/visualization.jpg",width=500)
    #st.subheader("About the Project")
    st.markdown("## :violet[A User-Friendly Tool Using Streamlit and Plotly]")

    st.markdown("##### Inspired from | https://www.phonepe.com/pulse/explore/transaction/2022/4/")
    st.markdown("## Learning outcomes & Process flow of this project:")
    st.write("### :violet[1.Data extraction and processing:]")
    st.write ("##### You will learn how to use Clone Github to extract data from a repository and pre-process the data using Python libraries such as Pandas.")
    st.write("### :violet[2.Data transformation:]")
    st.write ("##### Use a scripting language such as Python, along with libraries such as Pandas, to manipulate and pre-process the data. This may include cleaning the data, handling missing values, and transforming the data into a format suitable for analysis and visualization.")
    st.write("### :violet[3.Database insertion:]")
    st.write ("##### Use the 'postgresql-connector-python' library in Python to connect to a Postgres SQL database and insert the transformed data using SQL commands.")
    st.write("### :violet[4.Dashboard creation:]")
    st.write ("##### Use the Streamlit and Plotly libraries in Python to create an interactive and visually appealing dashboard. Plotly's built-in geo map functions can be used to display the data on a map and Streamlit can be used to create a user-friendly interface with multiple dropdown options for users toselect different facts and figures to display.")
    st.write("### :violet[5.Data retrieval:]")
    st.write ("##### Use the 'postgres-connector-python' library to connect to the PostgreSQL database and fetch the data into a Pandas dataframe. Use the data in the dataframe to update the dashboard dynamically.")
    st.write("### :violet[6.Deployment:]")
    st.write ("##### Ensure the solution is secure, efficient, and user-friendly. Test the solution thoroughly and deploy the dashboard publicly, making it accessible to users.")
    st.markdown("### Skills Take Away")
    st.markdown("# :violet[Data Visualization and Exploration]")
    
    st.write(" ")
    st.write(" ")
    st.markdown("#### :violet[Domain :] Fintech")
    st.markdown("#### :violet[Technologies used :] Github Cloning, Python, Pandas, PostresSQL, postressql-connector-python, Streamlit, and Plotly.")
    col1,col2,col3,col4,col5=st.columns(5,gap="small")
    with col1:

        st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/github.png")
    with col2:

        st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/python.png")

    with col3:

        st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/postgres.png")

    with col4:

        st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/streamlit.png")

    with col5:

        st.image("C:/Users/MY COMPUTER/Desktop/Phone pe project/plotly.png")


    st.markdown("#### :violet[Overview :] In this streamlit web app you can visualize the phonepe pulse data and gain lot of insights on transactions, number of users, top 10 state, district, pincode and which brand has most number of users and so on. Bar charts, Pie charts and Geo map visualization are used to get some insights.")




elif select == "Data Visulalization":

    tab1, tab2, tab3 = st.tabs(["Aggregated Ananlysis","Map Analysis","Top Analysis"])
    #select=option_menu("Data Exploration",["Aggregated Ananlysis","Map Analysis","Top Analysis"])

    with tab1:
        method1=st.radio("select the method",["Aggregated Transaction","Aggregated User"])

        if method1== "Aggregated Transaction":
            years=st.slider("select year",Aggre_transaction["Years"].min(),Aggre_transaction["Years"].max(),Aggre_transaction["Years"].min())
            tac_Y= Transaction_amount_count_Y(Aggre_transaction, years)

            quarters=st.slider("select Quarter",tac_Y["Quarter"].min(),tac_Y["Quarter"].max(),tac_Y["Quarter"].min())
            Transaction_amount_count_Y_Q(tac_Y, quarters)
            
            states=st.selectbox("select the states",tac_Y["States"].unique())
            Aggre_Trans_Transaction_Type(tac_Y, states)


        elif method1 == "Aggregated User":
            
            years=st.slider("select year",Aggre_user["Years"].min(),Aggre_user["Years"].max(),Aggre_user["Years"].min())
            Aggre_user_Y= Aggre_user_plot_1(Aggre_user, years)

            quarters=st.slider("select Quarter",Aggre_user_Y["Quarter"].min(),Aggre_user_Y["Quarter"].max(),Aggre_user_Y["Quarter"].min())
            Aggre_user_Y_Q=Aggre_user_plot_2(Aggre_user_Y, quarters)

            states=st.selectbox("select the states",Aggre_user_Y_Q["States"].unique())
            Aggre_user_plot_3(Aggre_user_Y_Q, states)

    with tab2:
        method2=st.radio("select the method",["Map Transaction","Map User"])

        if method2 == "Map Transaction":
        
            years=st.slider("select the year",Map_transaction["Years"].min(),Map_transaction["Years"].max(),Map_transaction["Years"].min())
            Map_tac_Y= Transaction_amount_count_Y(Map_transaction, years)

            states=st.selectbox("select the states_mt",Map_tac_Y["States"].unique())
            Map_Trans_District(Map_tac_Y, states)

            quarters=st.slider("select Quarter_mt",Map_tac_Y["Quarter"].min(),Map_tac_Y["Quarter"].max(),Map_tac_Y["Quarter"].min())
            Map_Trans_amount_count_Y_Q=Transaction_amount_count_Y_Q(Map_tac_Y, quarters)
            

        elif method2== "Map User":
            years=st.slider("select the year_mu",Map_user["Years"].min(),Map_user["Years"].max(),Map_user["Years"].min())
            Map_user_Y = Map_user_plot_1(Map_user, years)

            quarters=st.slider("select Quarter_mu",Map_user_Y["Quarter"].min(),Map_user_Y["Quarter"].max(),Map_user_Y["Quarter"].min())
            map_user_Y_Q=Map_user_plot_2(Map_user_Y, quarters)

            states=st.selectbox("select the states_mu",map_user_Y_Q["States"].unique())
            Map_user_plot_3(map_user_Y_Q,states)
                    

    with tab3:
        method3=st.radio("select the method",["Top Transaction","Top User"])

        if method3 == "Top Transaction":
            years=st.slider("select the year_tt",Top_transaction["Years"].min(),Top_transaction["Years"].max(),Top_transaction["Years"].min())
            Top_tac_Y= Transaction_amount_count_Y(Top_transaction, years)

            states=st.selectbox("select the states_tt",Top_tac_Y["States"].unique())
            Top_trans_plot_1(Top_tac_Y,states)

            quarters=st.slider("select Quarter_tt",Top_tac_Y["Quarter"].min(),Top_tac_Y["Quarter"].max(),Top_tac_Y["Quarter"].min())
            Top_Trans_amount_count_Y_Q=Transaction_amount_count_Y_Q(Top_tac_Y, quarters)

        elif method3== "Top User":
            years=st.slider("select the year_tu",Top_user["Years"].min(),Top_user["Years"].max(),Top_user["Years"].min())
            Top_user_Y=Top_user_plot_1(Top_user, years)

            states=st.selectbox("select the states_tu",Top_user_Y["States"].unique())
            Top_user_plot_2(Top_user_Y, states)




elif select == "Top Charts":

    Question=st.selectbox("select the Question",["1.Transation Amount and Count of Aggregated Transaction",
                                                "2.Transation Amount and Count Map Transaction",
                                                "3.Transation Amount and Count Top Transaction",
                                                "4.Transaction Count of Aggregated User",
                                                "5.Registered users of Map User",
                                                "6.App opens of Map User",
                                                "7.Registered users of Top User",
                                                "8.Transaction type and transaction count analysis of Aggregated Transaction",
                                                "9.Transaction type and transaction amount analysis of Aggregated Transaction"])



    if Question=="1.Transation Amount and Count of Aggregated Transaction":
        
        st.subheader("Transaction Amount")
        top_chart_transaction_amount("aggregated_transaction")
        st.subheader("Transaction Count")
        top_chart_transaction_count("aggregated_transaction")

    elif Question=="2.Transation Amount and Count Map Transaction":
        
        st.subheader("Transaction Amount")
        top_chart_transaction_amount("map_transaction")
        st.subheader("Transaction Count")
        top_chart_transaction_count("map_transaction")


    elif Question=="3.Transation Amount and Count Top Transaction":
        
        st.subheader("Transaction Amount")
        top_chart_transaction_amount("top_transaction")
        st.subheader("Transaction Count")
        top_chart_transaction_count("top_transaction")



    elif Question=="4.Transaction Count of Aggregated User":
        
        st.subheader("Transaction Count")
        top_chart_transaction_count("aggregated_user")


    elif Question=="5.Registered users of Map User":

        state=st.selectbox("select the state",Map_user["States"].unique())
        st.subheader("Registered Users")
        top_chart_registered_users("map_user",state)

    elif Question=="6.App opens of Map User":

        state=st.selectbox("select the state",Map_user["States"].unique())
        st.subheader("Appopens")
        top_chart_registered_users("map_user",state)

    elif Question=="7.Registered users of Top User":

        st.subheader("Registeredusers")
        top_chart_registered_user("top_user")


    elif Question=="8.Transaction type and transaction count analysis of Aggregated Transaction":
        
        st.subheader("Transaction Count Analysis")
        top_chart_transaction_type_count("aggregated_transaction")

    elif Question=="9.Transaction type and transaction amount analysis of Aggregated Transaction":

        st.subheader("Transaction Amount Analysis")
        top_chart_transaction_type_amount("aggregated_transaction")














    


