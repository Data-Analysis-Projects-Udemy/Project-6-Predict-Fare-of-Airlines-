# Project-6-Predict-Fare-of-Airlines-

 
0. [Instalar xlrd ](#schema0)

# a.-Comprensión de datos y Preprocesamiento de datos
1. [Cargar los datos y ver los nulos](#schema1)
2. [Limpiar los datos](#schema2)
3. [Separar las horas y los minutos de duración de la duración](#schema3)
4. [Separa las horas y los minutos](#schema4)
<hr>

<a name="schema0"></a>

# 0. Instalar xlrd
~~~python
conda install -c anaconda xlrd
~~~
Para poder leer archivos `xlsx`
<hr>

<a name="schema1"></a>

# 1. Cargar los datos y ver los nulos

~~~python
train_data=pd.read_excel('./data/Data_Train.xlsx')
~~~
![img](./images/001.png)


En este caso los podemos eliminar porque tenemos bastantes datos y solo dos son nulos.
![img](./images/004.png)
~~~python
train_data.dropna(inplace = True)
~~~
<hr>

<a name="schema2"></a>

# 2. Limpiar los datos
Convertir `Data_of_Journey, Dep_Time, Arrival_Time` a datetime, lo hacemos creando una función
~~~python
def change_into_datetime(col):
    train_data[col] = pd.to_datetime(train_data[col])
for time in ['Date_of_Journey','Dep_Time', 'Arrival_Time']:
    change_into_datetime(time)
~~~

![img](./images/005.png)

Convertir `Date_of_journey` a día y mes

~~~python
train_data['journey_day'] = train_data['Date_of_Journey'].dt.day
train_data['journey_month'] = train_data['Date_of_Journey'].dt.month
~~~
Ahora que tenemos los días y los meses en columnas separadas podemos borrar la columna `Date_of_Journey`

~~~python
train_data.drop('Date_of_Journey',axis = 1, inplace = True)
~~~

Convertir `Dep_Time` y `Arrival_Time` ahora
~~~python
def extract_hour(df,col):
    df[col + 'hour'] = df[col].dt.hour
    
    
def extract_min(df,col):
    df[col + 'minute'] = df[col].dt.minute

    
def drop_col(df,col):
    df.drop(col, axis = 1, inplace = True)

change = ['Dep_Time', 'Arrival_Time']

for col in change:
    
    extract_hour(train_data, col)
    extract_min(train_data, col)
    drop_col(train_data, col)
~~~

![img](./images/003.png)

<hr>

<a name="schema3"></a>

# 3. Separar las horas y los minutos de duración de la duración

~~~python
duration=list(train_data['Duration'])

for i in range(len(duration)):
    if len(duration[i].split(' '))==2:
        pass
    else:
        if 'h' in duration[i]:                  
            duration[i]=duration[i] + ' 0m'      
        else:
            duration[i]='0h '+ duration[i]   
train_data['Duration']=duration
~~~
![img](./images/006.png)

<hr>

<a name="schema4"></a>

# 4. Separa las horas y los minutos
~~~python
def hour(x):
    return x.split(' ')[0][0:-1]

def minute(x):
    return x.split(' ')[1][0:-1]


train_data['Duration_hours']=train_data['Duration'].apply(hour)
train_data['Duration_mins']=train_data['Duration'].apply(minute)
~~~
![img](./images/007.png)


Convertir las horas y los minutos a enteros
~~~python
train_data['Duration_hours']=train_data['Duration_hours'].astype(int)
train_data['Duration_mins']=train_data['Duration_mins'].astype(int)
~~~
![img](./images/008.png)

<hr>

<a name="schema5"></a>

# 5. Manejar datos categóricos y características de codificación

Manejo de datos categóricos
Estamos utilizando 2 técnicas de codificación principales para convertir datos categóricos en algún formato numérico.

Datos nominales -> los datos no están en ningún orden -> OneHotEncoder se usa en este caso
Datos ordinales -> los datos están en orden -> LabelEncoder se usa en este caso



Aerolínea vs Análisis de precios
~~~python
plt.figure(figsize = (15,5))
sns.boxplot(x = 'Airline', y = 'Price', data = train_data.sort_values('Price',ascending = False))
plt.savefig("./images/airline.png")
~~~
![img](./images/airline.png)

Conclusión -> En el gráfico podemos ver que Jet Airways Business tiene el precio más alto. Aparte de la primera aerolínea, casi todas tienen una mediana similar.

Realizar Total_Stops vs Análisis de precios

~~~python
plt.figure(figsize=(15,5))
sns.boxplot(y='Price',x='Total_Stops',data=train_data.sort_values('Price',ascending=False))
plt.savefig("./images/price.png")
~~~
![img](./images/price.png)


Como la aerolínea es un dato categórico nominal, realizaremos OneHotEncoding
~~~python
Airline=pd.get_dummies(categorical['Airline'], drop_first=True)
~~~
![img](./images/010.png)

Source vs Price

~~~python
plt.figure(figsize=(15,5))
sns.catplot(y='Price',x='Source',data=train_data.sort_values('Price',ascending=False),kind='boxen')
plt.savefig("./images/source.png")
~~~
![img](./images/source.png)

~~~python
Source=pd.get_dummies(categorical['Source'], drop_first=True)
~~~
![img](./images/011.png)

~~~python
plt.figure(figsize=(15,5))
sns.catplot(y='Price',x='Destination',data=train_data.sort_values('Price',ascending=False),kind='boxen')
plt.savefig("./images/destination.png")
~~~
![img](./images/destination.png)

~~~python
Destination=pd.get_dummies(categorical['Destination'], drop_first=True)
~~~
![img](./images/012.png)

~~~python
categorical['Route_1']=categorical['Route'].str.split('→').str[0]
categorical['Route_2']=categorical['Route'].str.split('→').str[1]
categorical['Route_3']=categorical['Route'].str.split('→').str[2]
categorical['Route_4']=categorical['Route'].str.split('→').str[3]
categorical['Route_5']=categorical['Route'].str.split('→').str[4]
~~~
![img](./images/013.png)


Cambiamos los valores `Nan`por `None` en la columna `Route`
~~~python
categorical.isnull().sum()
for i in ['Route_3', 'Route_4', 'Route_5']:
    categorical[i].fillna('None',inplace=True)
~~~







datos: https://drive.google.com/drive/folders/1QPizxIdGZ7TA9ecSGjDxJFLYNUOC8Kn9