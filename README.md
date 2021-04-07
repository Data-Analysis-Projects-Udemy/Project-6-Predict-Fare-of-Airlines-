# Project 6 Predict Fare of Airlines 

 
0. [Instalar xlrd ](#schema0)

# a.-Comprensión de datos y Preprocesamiento de datos
1. [Cargar los datos y ver los nulos](#schema1)
2. [Limpiar los datos](#schema2)
3. [Separar las horas y los minutos de duración de la duración](#schema3)
4. [Separa las horas y los minutos](#schema4)
5. [Manejar datos categóricos y características de codificación](#schema5)
6. [Detección de outlier](#schema6)
7. [Seleccione las mejores funciones usando Técnica de selección de funciones](#schema7)
8. [Aplicar la selección de características en los datos](#schema8)

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
Extraer ahora cuántas categorías hay en cada cat_feature
~~~python
for feature in categorical.columns:
    print('{} has total {} categories \n'.format(feature,len(categorical[feature].value_counts())))
~~~
![img](./images/014.png)

Como veremos, tenemos muchas funciones en Route, una codificación en caliente no será una mejor opción, permite aplicar la codificación de etiquetas.

~~~python
from sklearn.preprocessing import LabelEncoder
encoder=LabelEncoder()
for i in ['Route_1', 'Route_2', 'Route_3', 'Route_4','Route_5']:
    categorical[i]=encoder.fit_transform(categorical[i])
~~~


Cambiamos lo valores de la columna `Total_Stops`

~~~python
dict={'non-stop':0, '2 stops':2, '1 stop':1, '3 stops':3, '4 stops':4}
categorical['Total_Stops']=categorical['Total_Stops'].map(dict)
~~~
![img](./images/015.png)

Concatenamos dataframe --> categorical + Airline + Source + Destination
~~~python
data_train=pd.concat([categorical,Airline,Source,Destination,train_data[cont_col]],axis=1)
data_train.drop(columns=['Airline', 'Source','Destination'], inplace = True)
~~~
![img](./images/017.png)

<hr>

<a name="schema6"></a>

# 6. Detección de outlier

Función para dibujar
~~~python
def plot(df,col):
    fig,(ax1,ax2)=plt.subplots(2,1)
    sns.distplot(df[col],ax=ax1)
    sns.boxplot(df[col],ax=ax2)
plt.figure(figsize=(30,20))
plot(data_train,'Price')
plt.savefig("./images/out.png")
~~~
![img](./images/out.png)


Nos quedamos con los valores por debajo de `40000`
~~~python
data_train['Price']=np.where(data_train['Price']>=40000,data_train['Price'].median(),data_train['Price'])
plt.figure(figsize=(30,20))
plot(data_train,'Price')
plt.savefig("./images/no_out.png")
~~~
![img](./images/no_out.png)


<hr>

<a name="schema7"></a>

# 7. Seleccione las mejores funciones usando Técnica de selección de funciones

Separar tus datos independientes y dependientes
~~~python
X=data_train.drop('Price',axis=1)
X.head()
~~~

![img](./images/018.png)

~~~python
y=data_train['Price']
y
~~~
![img](./images/019.png)
<hr>

<a name="schema8"></a>

# 8. Aplicar la selección de características en los datos

~~~python
from sklearn.feature_selection import mutual_info_classif
mutual_info_classif(X,y)
~~~
![img](./images/020.png)
~~~python
imp=pd.DataFrame(mutual_info_classif(X,y),index=X.columns)
imp
~~~

![img](./images/021.png)

~~~python
imp.columns = ['importance']
imp.sort_values(by = 'importance', ascending = False)
~~~
![img](./images/022.png)




















datos: https://drive.google.com/drive/folders/1QPizxIdGZ7TA9ecSGjDxJFLYNUOC8Kn9