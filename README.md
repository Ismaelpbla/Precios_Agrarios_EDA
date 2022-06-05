
# EXPLORATORY DATA ANALYSIS ABOUT FRUIT PRICES IN THE VALENCIA PROVINCE (SPAIN)

The main objective of this project is undergoing an exploratory data analysis of the mean prices of some of the most common fruits in the Valencia Province.
The motivation behind this project is to know how interesting will be to invest or not in different kinds of fruit in the Valencia territory.




## Authors

- [@Ismael P. Blanquer](https://www.github.com/Ismaelpbla)


## DATA

The data has extracted by Web Scrapping using the library Beatiful Soup, from two official websites of the Valencia Government:

- [Agriculture prices]('https://agroambient.gva.es/es/precios-agrarios'), published by "Consellería de Agricultura y Medio Ambiente de la Comunidad Valenciana"

- [Registro General de Parcelas Agrarias ("REGEPA")]('https://agroambient.gva.es/es/estadistiques-agricoles') also published by "Consellería de Agricultura y Medio Ambiente de la Comunidad Valenciana"


## DATA CLEANING

The process of data cleaning was the following.

1. We join all the dataframes from the 2016 to 2022 in one dataframes
2. Before join we add a column that identify the year of each dataset

```
#p2016 = pd.read_csv('precio2016.csv') # leemos los archivos csv obtenidos mediante el webscraping
p2017 = pd.read_csv('precio2017.csv')
p2018 = pd.read_csv('precio2018.csv')
p2019 = pd.read_csv('precio2019.csv')
p2020 = pd.read_csv('precio2020.csv')
p2021 = pd.read_csv('precio2021.csv')
p2022 = pd.read_csv('precio2022.csv')

#p2016['año'] = 2016 # Añadimos la columna de año a cada dataframe
p2017['año'] = 2017
p2018['año'] = 2018
p2019['año'] = 2019
p2020['año'] = 2020
p2021['año'] = 2021
p2022['año'] = 2022

precios = pd.concat([p2017,p2018,p2019,p2020,p2021,p2022]) # creamos un nuevo datframe con todos los precios de todos los años

```
3. We select only the regular fruit, the citrus fruit, and the nuts.

```
# Creamos 3 dataframes: Uno con los citricos, otro con las frutas, y otro con los frutos secos

precios_citricos = precios.loc[(precios['producto'] == '01 Cítricos') | (precios['producto'] == '01 Citrics'), :] 

precios_frutas = precios.loc[(precios['producto'] == '02 Frutales') | (precios['producto'] == '02 Fruiters'), :]

precios_fsecos = precios.loc[(precios['producto'] == '04 Frutos secos') | (precios['producto'] == '04 Fruita Seca'),:]

precios_citricos

```

4. We use two functions to clean the dataframe 

```
def precios_mes (precios_y, subtipo): # función creada para limpiar los datos de los dataframes y obtener los datos de un subtipo concreto 

    precios_x = precios_y.loc[(precios_y['subtipo'] == subtipo), :] # Nos quedamos únicamente con las filas de ese subtipo

    precios_x['precio_min'] = precios_x.precio_min.str.replace('€/kg', '0') # Sustituimos las celdas con el string '€/Kg' por un 0

    precios_x['precio_min'] = precios_x.precio_min.str.replace('€/Kg', '0') 

    precios_x['precio_min'] = precios_x.precio_min.str.replace(',', '.') # Sustituimos los puntos por comas

    precios_x['precio_min'] = precios_x.precio_min.astype('float64') # pasamos el precio minimo a float

    precios_x['precio_max'] = precios_x.precio_max.str.replace('€/kg', '0')

    precios_x['precio_max'] = precios_x.precio_max.str.replace('€/Kg', '0')

    precios_x['precio_max'] = precios_x.precio_max.str.replace(',', '.')

    precios_x['precio_max'] = precios_x.precio_max.astype('float64')

    precios_x['año'] = precios_x.año.astype('int64') # pasamos el año a entero

    precios_x['mes'] = precios_x.mes.str.replace('enero', '1') # Sustituimos cada mes por su numero correspondiente
    precios_x['mes'] = precios_x.mes.str.replace('febrero', '2')
    precios_x['mes'] = precios_x.mes.str.replace('marzo', '3')
    precios_x['mes'] = precios_x.mes.str.replace('abril', '4')
    precios_x['mes'] = precios_x.mes.str.replace('mayo', '5')
    precios_x['mes'] = precios_x.mes.str.replace('junio', '6')
    precios_x['mes'] = precios_x.mes.str.replace('julio', '7')
    precios_x['mes'] = precios_x.mes.str.replace('agosto', '8')
    precios_x['mes'] = precios_x.mes.str.replace('septiembre', '9')
    precios_x['mes'] = precios_x.mes.str.replace('octubre', '10')
    precios_x['mes'] = precios_x.mes.str.replace('noviembre', '11')
    precios_x['mes'] = precios_x.mes.str.replace('diciembre', '12')

    precios_x['mes'] = precios_x.mes.astype('int64') # pasamos el mes a entero
    precios_x['semana'] = precios_x.semana.str.lstrip('Semana+') # Eliminamos el string 'Semana+' y aislamos el numero de la semana
    precios_x['semana'] = precios_x['semana'].astype('int64') # pasamos la semana a entero

    precios_x_2 = precios_x.copy() # creamos una copia de lo que llevamos hecho

    precios_x_2 = precios_x_2.rename(columns={'año':'year', 'mes':'month'}) #renombramos el año y el mes por year y month

    precios_x_2['date_semana'] = precios_x_2.apply(lambda x: str(x.year)+"-"+str(x.semana), axis=1) #creamos una columna date_semana con la fecha en formato año-semana

    precios_mes = precios_x_2.groupby('date_semana').mean().reset_index() # agrupamos por date_semana 

    return precios_x_2 # Nos devuelve el dataframe agrupado por date_semana
    
```
5. Sometimes the names of the fruits were in Valencian lenguage, so we change it to be in always in the same language, in this case Spanish.

One exemple with tangerines:

```
# Cambiamos los nombres de las mandarinas para tener  solamente Satsumas, Clementinas y Otras Mandarinas

precios_citricos['subtipo'] =  precios_citricos.subtipo.str.replace('02- Clementinas', 'Clementinas')
precios_citricos['subtipo'] =  precios_citricos.subtipo.str.replace('03- Otras Mandarinas', 'Otras Mandarinas')
precios_citricos['subtipo'] =  precios_citricos.subtipo.str.replace('01- Satsumas', 'Satsumas')
precios_citricos['subtipo'] =  precios_citricos.subtipo.str.replace('02- Clementines', 'Clementinas')
precios_citricos['subtipo'] =  precios_citricos.subtipo.str.replace('03- Altres Mandarines', 'Otras Mandarinas')

```

6. Finally we apply the function to the initial dataframes and we obtain the mean prices of a specific fruit with date values.

An example with tangerines:

```
# Aplicamos la función de limpieza

precios_mandarinas_1 = precios_mes(precios_citricos, 'Clementinas')
precios_mandarinas_2 = precios_mes(precios_citricos, 'Satsumas')
precios_mandarinas_3 = precios_mes(precios_citricos, 'Otras Mandarinas')

# Y concatenamos

precios_mandarinas = pd.concat([precios_mandarinas_1, precios_mandarinas_2, precios_mandarinas_3])

```


## DATA VISUALIZATION

### General distribution of fruits crops in Valencia Province

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/superficies_totales.png?raw=true)

### Price variation through the week of the year

![img](https://raw.githubusercontent.com/Ismaelpbla/Precios_Agrarios_EDA/main/Figuras/precios_evolucion/precios_naranjas.png)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precios_mandarina.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precios_almendra.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_caqui.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_granada.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precios_higo.png)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precios_cereza.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_albaricoque.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_nispero.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_nectarina.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/precios_evolucion/precio_melocoton.png?raw=true)

### Most profitable varieties of Orange, Tangerine, Almond and summer fruit

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Distribuci%C3%B3n_precios/precios_medios_naranjas.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Distribuci%C3%B3n_precios/precios_medios_mandarinas.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Distribuci%C3%B3n_precios/almendra_precios.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Distribuci%C3%B3n_precios/fruta_verano.png?raw=true)

### Amount of Area of each oranges and tangerines crops in Valencia Province, as we can see there is a loss of area in tangerines and a gain in oranges

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/naranjas_superficie.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/super_mandarinas.png?raw=true)

### Counties with more area of oranges, tangerines, almonds, persimmons or grenades.

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/comarcas_naranjas.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/comarcas_mandarinas.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/comarcas_almendra.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/comarcas_caqui.png)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/comarcas_granada.png?raw=true)

### Municipalities with more area of oranges, tangerines, almonds, persimmons or grenades.

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/municipios_naranjas.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/municipios_mandarina.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/municipios_almendra.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/municipios_caquis.png?raw=true)
![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/Superficies/municipios_granadas.png?raw=true)

### Temporality of summer fruit with a Gantt Chart

![img](https://github.com/Ismaelpbla/Precios_Agrarios_EDA/blob/main/Figuras/temporalidad%20frutas.png?raw=true)

## DISCUSSION AND CONCLUSIONS

This project was a practical part of the Bridge Bootcamp of Data Science in Valencia. All the conclusions and discussions of the data were in the presentation of this project, also included in this repository.
Check the PDF presentation for more information.