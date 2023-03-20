# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย
## 1 : Datset
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
    
	Source :  https://data.thaiauto.or.th
  - ข้อมูลการผลิตรถยนต์ภายในประเทศไทย
  - ข้อมูลผลิตรถนยนต์เพื่อการส่งออกต่างประเทศ
  - สถิติรถจดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
  
1.2) สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้า ถึง ธค 2565 การไฟฟ้านครหลวง
     
	Source : https://data.go.th/dataset/mea-customer
		 
1.3) ข้อมูล DC EV Charging Station Thailand - Piyamate Wisanuvej.kml
    
	Source : http://bit.ly/401dOwX

1.4) ข้อมูลภูมิศาสตร์ระบุขอบเขตจังหวัดของประเทศไทย Thailand - Subnational Administrative Boundaries Dataframe 
     
	Source : https://data.humdata.org/dataset/cod-ab-tha

## 2 : Data Cleansing and EDA

2.1) Import Library & Import Data
```
#Install library
!pip install geopandas 
!pip install descartes 
!pip install contextily
!pip install mapclassify

#Import library
import sys
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import matplotlib as mpl
import seaborn as sns
import datetime as dt
import geopandas as gpd
from shapely.geometry import Point

plt.style.use('seaborn')
```
2.2) Data Ceansing and Transform process

2.2.1) ข้อมูลการผลิตรถยนต์ภายในประเทศไทยและข้อมูลผลิตรถนยนต์เพื่อการส่งออกรถยนต์ต่างประเทศ
```
df_prod_dom = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-production-thai.xlsx?raw=true",header=[0,1]) 
RangeIndex: 133 entries, 0 to 132
Data columns (total 19 columns):

df_export_unit = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-export-unit.xlsx?raw=true" ,header=[0,1]) 
RangeIndex: 132 entries, 0 to 131
Data columns (total 47 columns):
```
ลักษณะของข้อมูลที่พบและต้องจัดการคือ
	1. ส่วนของ dataframe header เป็น 2 ชั้น จึงต้องทำการรวบให้เหลือเพียง 1ชั้น 
	2. ชื่อเดือนมี white space ปนอยู่และตัวเลขใน dataframe มีจุลภาค(comma)ต้องกำจัดออกก่อนจะแปลงเป็น data type ที่ต้องการ
	3. เดือนเป็น ชื่อเดือนภาษาไทย ต้องทำการเปลี่ยนเป็นตัวเลขเพื่อให้สามารถแปลงวัน Datetime format แล้วสามารารถนำไปใช้งานต่อได้
```
df_prod_dom.columns = df_prod_dom.columns.map('_'.join)
df_prod_dom_1 = df_prod_dom.rename(columns={'Unnamed: 1_level_0_ปี': 'Year','Unnamed: 2_level_0_เดือน/ไตรมาส': 'Month'})
df_prod_dom_1 = df_prod_dom_1[:-1]
df_prod_dom_1 = df_prod_dom_1.drop('Unnamed: 0_level_0_ลำดับที่', axis=1)
```
```
df_export_unit.columns = df_export_unit.columns.map('_'.join)
df_export_unit = df_export_unit.rename(columns={'Unnamed: 1_level_0_ปี': 'Year','Unnamed: 2_level_0_เดือน/ไตรมาส': 'Month'})
df_export_unit = df_export_unit[:-1]
df_export_unit = df_export_unit.drop('Unnamed: 0_level_0_ลำดับที่', axis=1)
```
```
#Remove comma and space
df_export_unit1 = df_export_unit.replace(',','', regex=True)
# Remove white space
df_export_unit1['Month'] = df_export_unit1['Month'].str.replace(r"\s+", "")
```
```
month_map = {'มกราคม': 1, 'กุมภาพันธ์': 2, 'มีนาคม': 3, 'เมษายน': 4, 'พฤษภาคม': 5, 'มิถุนายน': 6, 'กรกฎาคม': 7, 'สิงหาคม': 8, 'กันยายน': 9, 'ตุลาคม': 10, 'พฤศจิกายน': 11, 'ธันวาคม': 12}

df_prod_dom_3['Month'] = df_prod_dom_3['Month'].map(month_map)

df_export_unit2['Month'] = df_export_unit2['Month'].map(month_map)
df_export_unit2['Year'] = df_export_unit2['Year'].astype(int)
```
2.2.2) ปริมาณรถยนต์จดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
ข้อมูลมีคุณภาพดี ไม่ต้องทำการ Cleansing ใดๆ
```
new_register = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/New_Car_with_Fuel.csv?raw=true")
new_register.info()
```
```
RangeIndex: 335 entries, 0 to 334
Data columns (total 12 columns):
```
2.2.3)สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้า ถึง ธ.ค. 2565 การไฟฟ้านครหลวง
ลักษณะของข้อมูลที่พบและต้องจัดการคือ
1. ข้อมูลมีแถวที่เป็น NaN ทั้งส่วนบนและส่วนล่างของ dataframe เนื่องจากข้อมูลเก็บมาในรูปแบบ Excel จึงต้องตัดออก และเลือกเฉพาะคอลัมน์ 'สถานีอัดประจุไฟฟ้า' มาใช้ในการวิเคราะห์
2. ข้อมูลที่ได้รับมาอยู่ในรูปปี พุทธศักราช(B.E.) ส่งผลให้ไม่สามารถ แปลงข้อมูลให้อยู่ในรูปของ datetime format ได้ จึงต้องทำการแปลงให้เป็นคริสต์ศักราช (A.D.)
```
charger_df = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EV_Charger(x-).csv?raw=true",header = 2)
charger_df.info()
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 55 entries, 0 to 54
Data columns (total 52 columns):
```
แทนค่า NaN ด้วย 0 และตัดแถวกับคอลัมน์ที่ไม่ต้องการออก
```
charger_df1 = charger_df.iloc[2:50, [0,1,11]]
charger_df1 
charger_df1.fillna(0)
charger_df1['สถานีอัดประจุไฟฟ้า'] = charger_df1['สถานีอัดประจุไฟฟ้า'].astype(int)
```
เปลี่ยนเลขปีเป็น ค.ศ.และแปลงเป็น datetime format
```
charger_df7 = charger_df6.rename(columns={'ปี': 'Year','เดือน':'Month'})
charger_df7['Year'] = charger_df7['Year']-543
charger_df7.insert(2, 'Year-Month', pd.to_datetime(charger_df7[['Year', 'Month']].assign(Day=1)) )
charger_df7
```
2.2.4)ข้อมูลภูมิศาสตร์ระบุขอบเขตจังหวัดของประเทศไทยและ DC EV Charging Station Thailand

```
#Import Thailand map data
#Thailand - Subnational Administrative Boundaries Dataframe from https://data.humdata.org/dataset/cod-ab-tha
gdf = gpd.read_file('tha_admbnda_adm1_rtsd_20190221.shp')
gdf.head()
```
ลักษณะของข้อมูลที่พบและต้องจัดการคือ
1. เนื่องจากไฟล์ DC EV Charging Station Thailand - Piyamate Wisanuvej เป็น kml file จึงนำไปแปลงเป็น csv file ด้วยเว็ป https://mygeodata.cloud/converter/ ก่อน ได้ออกมาทั้งหมด 7 ไฟล์ตามประเภทของผู้ให้บริการแล้วจึงต้องทำการรวมไฟล์ทั้ง 7 ไฟล์เป็น Dataframe เดียว
2. DC EV Charging Station Thailand ข้อมูลที่ตั้งเป็นภาษาไทยและยากที่จะแยกเอาจังหวัดออกมา จึงใช้วิธีการเอาข้อมูล Latitude และ Longitude ไปเทียบกับ Polygon geometry เพื่อเอาชื่อจังหวัดจากแผนที่มาเพิ่มข้อมูลของจังหวัดของที่ตั้งสถานีก่อนนำมาวิเคราะห์
```
#Read all data chager locations of each brand.
df_ea24 = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EA_24hr.csv?raw=true')
df_eaop= pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EA_Off_peak.csv?raw=true')
df_elex = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EleX.csv?raw=true')
df_evolt = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EVolt_Sharge_etc.csv?raw=true')
df_mg = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/MG.csv?raw=true')
df_pea = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/PEA.csv?raw=true')
df_ptt = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/PTT.csv?raw=true')
df_charger = pd.concat([df_ea24,df_eaop,df_elex,df_evolt,df_mg,df_pea,df_ptt])
```
```
#Mapping Charging station point with province name by mapping Latitude and Longitude data with polygon data in GeodataFrame
def get_province_name(lat, lon, gdf):
    point = Point(lon, lat)
    for index, row in gdf.iterrows():
        if point.within(row.geometry):
            return row['ADM1_EN']
    return 'Not found'
    
df_charger['province_name'] = df_charger.apply(lambda row: get_province_name(row['Latitude'], 
                                                                             row['Longitude'], gdf), axis=1)
```
## 3 : EDA
3.1)	ปริมาณการผลิตยานยนต์สำหรับในประเทศ

ทำการ Sum ข้อมูล ของแต่ละปีเพื่อนำมาพลอต bar chart ดูแนวโน้มของปริมาณการผลิตในแต่ละปีและเปรียบเทียบสัดส่วนของ Passenger car กับ Pickup 1 Ton

```
df_plot = df_plot1.groupby('Year').sum().reset_index()
ax = df_plot.set_index('Year').plot(
    kind='bar', stacked=True, figsize=(20, 10))

# หัวตาราง
plt.title('\nDomestic Production (Units)\n', fontsize=25)
# ชื่อแกน x,y
plt.xlabel('Year', fontsize=20)
plt.ylabel('mils Unit', fontsize=20)
# หมุน ชื่อแกน x
plt.xticks(rotation=0, fontsize=15)
plt.yticks(fontsize=15)
# ขนาดคำอธิบาย
plt.legend(fontsize='15')
```
![image](https://user-images.githubusercontent.com/114766023/226316032-e52da559-bd71-459a-abc9-7b8fc65129b0.png)

```
plt.figure(figsize=(15,5),dpi=150)
ax = plt.gca()
df_plot.plot.bar(ax=ax, x='Year',title='Domestic Production (Units)', ylabel='The number of players', xlabel='Year')
plt.xticks(rotation=1)
```
![image](https://user-images.githubusercontent.com/114766023/226315791-6326c04e-1391-444a-8b77-da0c70d96ea6.png)

เพื่อจะดูแนวโน้มของของ Non-commercialcar ในแต่ละเดือนจึงรวมปริมาณขอ Passenger car กับ Pickup 1 Ton ก่อน แล้วจึงพลอต Line chart และเพิ่มเหตุการณ์สำคัญที่น่าจะมี
ผลต่ออุตสากรรมทเข้าไปในกราฟ
```
df_plot3 = df_plot1
df_plot3['Total Non-Comercial'] = df_plot3.apply(lambda x: x['Passenger Car_Sub Total'] + x['Pickup 1 Ton_Sub Total'] , axis=1)
df_plot3
```
```
plt.figure(figsize=(15,5), dpi=150)

# (Optional) Set the format of datetime displayed in x-axis
ax = plt.gca()
formatter = mpl.dates.DateFormatter('%Y')
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y'))



ax.set_xlim(dt.datetime(2012, 1, 1), dt.datetime(2023, 1, 1))
# xlabel
# major tic


# Alternative 1: Plot with the given x and y
plt.plot(df_plot1['Year-Month'], df_plot3['Total Non-Comercial'],    # x and y to plot
         color='midnightblue', marker='o', linestyle='solid')    # The matplotlib linestyle )
ax.set_title('Non-commercial car production history past 10 Yrs')
ax.annotate('The first-car tax scheme End', xy=(pd.Timestamp('2013-01-01'), 220000),
            xytext=(pd.Timestamp('2012-05-01'), 100000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
            
ax.annotate('Covid-19:Lock down', xy=(pd.Timestamp('2020-04-01'), 24000),
            xytext=(pd.Timestamp('2018-01-01'), 23000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
ax.annotate('2014 Thai coup détat', xy=(pd.Timestamp('2014-06-01'), 145000),
            xytext=(pd.Timestamp('2014-02-01'), 60000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
ax.annotate('Covid-19:Lock down', xy=(pd.Timestamp('2020-04-01'), 24000),
            xytext=(pd.Timestamp('2018-01-01'), 23000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )     
```
![image](https://user-images.githubusercontent.com/114766023/226317965-781dc0bd-a93e-49c0-8f9e-12296d83fa7a.png)

ปรับช่วงข้อมูลให้มาอยู่ในช่วง 5 ปีย้อนหลัง เพื่อให้เห็นแนวโน้มชัดขึ้น
![image](https://user-images.githubusercontent.com/114766023/226318066-9709ee02-6552-42e9-86a0-30344d163ab7.png)

3.2) ข้อมูลผลิตรถนยนต์เพื่อการส่งออกต่างประเทศ

เนื่องจากสนใขเแพาะ non-comnmercial เท่านั้น จึงต้องสร้าง column ผลรวมของ non-comnmercial car (Passener + Pickup ruck + PPV) ของแต่ภูมิภาคขึ้นมาจากก่อน 
แล้วจึงลองพลอตด้วย line chart
```
col_d  = []
for cont in continents:
    passenger_col = f'{cont}_Passenger Car'
    truck_col = f'{cont}_Pick up'
    ppv_col = f'{cont}_PPV'
    total_col = f'{cont}_Total'
    other_col = f'{cont}_Other'
    col_d.append(total_col)
    col_d.append(other_col )
    result_col = df_export_unit2[passenger_col] + df_export_unit2[truck_col] + df_export_unit2[ppv_col]
    df_export_unit2[f'Total_Non_com_{cont}'] = result_col   
df_export_dropped = df_export_unit2.drop(col_d, axis=1)
df_plot_export_group = df_export_dropped.drop(df_export_dropped.iloc[:,3:32], axis=1)
df_plot_export_group1 = df_plot_export_group.drop(df_plot_export_group.iloc[:,0:2], axis=1)
df_plot_export_group1
```

```
plt.figure(figsize=(10,5),dpi=150)
ax = plt.gca()
df_plot_export_group1.plot(kind='line',ax=ax, x='Year-Month', ylabel='Units', xlabel='Year',style='.-')
plt.xticks(rotation=1)
ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.2), ncol=3)
ax.set_xlabel('Year')
ax.set_title('Total Non-commercial car production history past 10 Years by Region')
```
![image](https://user-images.githubusercontent.com/114766023/226210595-15079ccd-2325-4c31-b31a-ef57f9b1be1d.png)

เนื่องจากกราฟมีจำนวนหลายเส้นและตัดกันจึงทำการแยกกราฟของแต่ละภูมิภาคออกเป็น subplot เพื่อให้เห็นแนวโน้มที่ชัดเจนขึ้น

![image](https://user-images.githubusercontent.com/114766023/226210568-c4db94dc-d0f5-4178-857b-72a77bdc1f42.png)


3.3) ปริมาณรถยนต์จดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
ดูแนวโน้มของรถยนต์จดทะเบียนใหม่ย้อนหลัง 5 ปี และดูสัดส่วนของรถยนต์แต่ละประเภทเชื้อเพลิงในแต่บะปีด้วย Stack bar chart
(ICE =  internal combustion engine, PHEV = Plug-in Hybrid Electic vehicle, HERV = Hybrid Electric Vehicle, BEV = Baterry Electric Vehicle)
ซึ่งพบว่ากราฟปี 2565 มีปริมาณเพิ่มขึ้นจากปี 2564 ซึ่งพ้นจากภาวะการระบาดของโควิด-19

```
new_register_df10 = new_register_df9.iloc[:3]
new_register_df11 =new_register_df10.loc[:, ['HEV', 'PHEV','BEV']]
new_register_df11

plt.figure(figsize=(15,5),dpi=150)
ax = plt.gca()
new_register_df11.plot( kind='barh', ax=ax,stacked=True,
         title='Number of xEV rigistry car', ylabel='The number of xEV rigistry car' )


left, right = ax.get_xlim()    
ax.set_xlim(left, right+0.7) 
```
![image](https://user-images.githubusercontent.com/114766023/226322538-15dd7df8-f317-43e4-b20d-5486f7739028.png)


![image](https://user-images.githubusercontent.com/114766023/226210712-13c23bec-82a0-4f45-96fd-510e5a7043bc.png)

เลือกดูในส่วนเฉพาะของ EV ว่ามีสัดส่วนเป้นเป็นอย่างไรในช่วง 3 ปีที่ผ่านมา นับตั้งแต่เกิดการระบาดของโควิด-19

![image](https://user-images.githubusercontent.com/114766023/226322871-4f7b4486-0347-4ec5-b027-860fbb6a32d0.png)

เทียบสัดส่วน 2564-2565 YoY
![image](https://user-images.githubusercontent.com/114766023/226323639-6300e468-d4e2-4597-89a1-f5ba846f2851.png)


เปรียบเทียบปริมาณรถ EV ที่จดทะเลียนใหม่แต่ละประเภท และดูอัตราการเติบโตเทียบกับปีก่อนหน้า (%YOY Growth rate)
![image](https://user-images.githubusercontent.com/114766023/226210815-94db3279-77a3-4fe5-855e-0e62673ff223.png)
![image](https://user-images.githubusercontent.com/114766023/226324759-7059ddf1-e77b-4af6-b5a1-8e02a6e9522d.png)


3.4) สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้า ถึง ธค 2565 การไฟฟ้านครหลวง ใช้ในการหาจำนวสนสถานีอัดประจุในพื้นที่ กทม.

คำนวนหาอัตราการเพิ่มขึ้นของสถานีอัดประจุจากปีก่อนหน้า (%YOY Growth rate)
```
diff = (charger_df4['สถานีอัดประจุไฟฟ้า'] - charger_df4['สถานีอัดประจุไฟฟ้า'].shift(1))/charger_df4['สถานีอัดประจุไฟฟ้า']

charger_df4['percent_growth'] = diff*100
charger_df4
```
![image](https://user-images.githubusercontent.com/114766023/226256039-244994bf-ddc7-49a8-bc2c-8334bb2f4962.png)

```
#Plot line chart to see the trend
plt.figure(figsize=(15,5), dpi=150)
plt.plot( charger_df7['Year-Month'], charger_df7['สถานีอัดประจุไฟฟ้า'], marker='o' )
plt.suptitle("Cumulative Quantity Charger Station in Bangkok")
```
จากกราฟแสดงให้เห็นว่าในช่วง 3 ปีย้อนหลังมานี้ จำนวนสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้าที่จดทะเบียนกับ กฟน. มีอัตราเพิ่มขึ้นทุกปีและโดยเฉพาะอย่างในปี 2565 ที่มีอัตราการ เติบโตจากปี 2564 ถึง 35% YoY

![image](https://user-images.githubusercontent.com/114766023/226325682-dc549937-4b99-4ec5-8314-9dd8976db1eb.png)
จากกราฟจะเห็นได้ว่าจำนวนสถานีอัดประจุไฟฟ้าเพิ่มขึ้นอย่างรวดเร็วในปี 2565 ตั้งแต่ช่วงไตรมาส 2 เป็นต้นไป สอดคล้องนโยบายของภาครัฐที่มีออกมาเพื่อสนับสนุนการใช้และผลิตรถยนต์ไฟฟ้าในประเทศไทย

3.5) ข้อมูลจำนวนสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้าประเภท DC ในประเทศไทย



## 4 : Question and Answer
ในการศึกษาครั้งนี้จะสนใจเฉพาะรถยนต์ที่ไม่ใช่เพื่อการพานิชย์ซึ่งก็คือรถยนต์นั่งส่วนบุคคล (Passenger car) รถกระบะขนาดด 1 ตัน (Pickup 1 ton truck) และรถกลุ่ม PPV เท่านั้น

คำถาม 1) สถานการณ์การผลิตรถยนต์ส่งออกทั่วโลกจากอดีตถึงปัจจุบันเป็นอย่างไร

คำตอบ ในที่นี้ใช้ข้อมูลยอดการผลิตเพื่อการส่งออกไปต่างประเทศของอุตสาหกรรมรถยนต์ไทยแทนมุมมองของภาวะตลาดรถยนต์โลก โดดยจากกราฟทำให้ทราบว่า

	1. ไทยส่งออกรถยนต์ไปตลาด เอเชียนเป็นอันดับหนึ่ง และอันดับสองคือออสเตรเลียเป็นอันดับ2
	2. หลังจากวิกฤตการระบาดของโรคโควิด-19(ปี 2020-21) พบว่ายอดการผลิตเพื่อส่งออกสู่ทุกภูมิภาคกลับมาเป็นขาขึ้นโดยเฉพาะอย่างยิ่งตลาดเอเชียและออสเตรเลียที่เพิ่มสูงขึ้นจนมากกว่าก่อนการระบาดของโควิด-19

	จึงสรุปได้ว่าการสถานการณ์การผลิตรถยนต์ส่งออกทั่วโลกที่ได้รับผลกระทบจากวิกฤตการระบาดของโรคโควิด-19 ฟื้นตัวกลับมาจนเข้าสู่ภาวะเหมือนก่อนการระบาดแล้ว
คำถาม	2) อุตสาหกรรมรถยนต์ในประเทศไทยยังสดใสหรือไม่

สมมติฐาน อุตสาหกรรมผลิตรถยนต์ในประเทศไทยฟื้นตัวและมีแนวโน้มเติบโตขึ้นหลังจากผ่านวิกฤตโควิด-19

คำตอบ 	ในที่นี้ใช้ข้อมูลยอดการผลิตสำหรับในประเทศแทนทิศทางของอุตสาหกรรมรถยนต์ภายในประเทศ เนื่องจากหากยอดสั่งซื้อ/ผลิตสูงขึ้นหรือลดลง อุตสาหกรรมและธุรกิจที่เกี่ยวข้องในประเทศก็จะได้รับผลกระทบและไปใน
ทิศทางเดียวกัน โดยจากข้อมูลและกราฟสรุปได้ว่า

1. อุตสาหกรรมรถยนต์ในประเทศมีการฟื้นตัวหลังจากวิกฤตการระบาดของโรคโควิด-19 แต่ยังไม่เท่ากับช่วงปี 2020 อาจจะด้วยปัจจัยจากสภาพเศรษฐกิจที่อยู่ในภาวะเงินเฟ้อซึ่งยังไม่สามารถระบุได้ชัดเจนด้วยข้อมูลที่มี


2. สัดส่วนการผลิต/ซื้อรถกระบะมีมากกว่ารถยนต์ส่วนบุคคล(รถเก๋ง)และสัดส่่วนของรถกระบะยังเพิ่มขึ้นอย่างต่อเนื่อง

คำถาม	3) รถยนต์ไฟฟ้าในประเทศประเทศไทยทิศทางและการเติบโตเป็นอย่างไร

สมมติฐาน จากนโบายของรัฐที่ออกมาส่งเสริมการใช้และผลิตรถไฟฟ้าในประเทศที่ออกมาปลายปี2565 
	ทำให้คาดว่ารถยนต์ไฟฟ้ามีการเติบโตของผู้ใช้ที่สูงขึ้นและมีทิศทางที่กินส่วนแบ่งของรถยนต์สันดาปภายในมากขึ้นเรื่อยๆ

คำตอบ 	จากการวิเคราะห์ยอดจดทะเบียนรถยนต์ใหม่ตั้งแต่ปี 2018-2022 พบว่าเป็นไปตามสมมติฐาน
	
1. โดยสัดส่วนของรถยนต์ไฟฟ้า(PHEV,HEV,BEV) รวมกันมีสัดส่วนเทียบกับรถยนต์สันดาปภายในขึ้นทุกปี รวมทั้งปริมาณรถยนต์ใหม่ก็เพิ่มขึ้นมาก 
โดยเห็นได้ชัดจากยอดจดทะเบียนปี 2021 ประมาณ 43,000 คันขึ้นมาเป็นเกือบ 83,000 คันหรือเกือบเท่าตัวเลยทีเดียว
	
2. จากสัดส่วนรถยนต์ไฟฟ้า(EV) แต่ละประเภทพบว่าสัดส่วนการเติบโตของรถยนต์ Baterry EV (BEV) 
มีเพิ่มมากขึ้นเมื่อเทียบกับ Hybrid EV (HEV) และ Plug-In Hybrid EV (PHEV)โดยสัดส่วน BEV เทียบกับ EV อื่นๆ 
เพิ่มขึ้นจาก 4.5% ในปี 2564 เป็น 11.4% ในปี 2565 และยอดจดทะเบียนปี 2565 เพิ่มขึ้นจาก ปี 2564 คิดเป็นอัตราการเติบโตสูงถึงกว่า 79% 
ในขณะที่ PHEV กับ HEV ถึงจะมียอดเพิ่มขึ้นแต่เทียบเป็นอัตราการเติบโตแล้วกลับพบว่าลดลงสวนทางกับ BEV 
จึงอาจจะกล่าวได้ว่าประเทศไทยกำลังเปลี่ยนผ่านสู่ยุครถยนต์ไฟฟ้าแบบใช้ไฟฟ้า 100% เช่น BEV 
และสัดส่วนของรถไฟฟ้ามีแนวโน้มจะกินส่วนแบ่งรถยนต์แสันดาปภายใน (Internal Combustion Engine - ICE) 
เพิ่มขึ้นทุกปีตามมาตราการที่รัฐส่งเสริมทั้งฝั่งผู้บริโดภคคือการลดภาษีและฝั่งผู้ผลิตที่ต้องผลิตรถไฟฟ้าให้มากกว่ายอดที่นำเข้าทั้งคัน (CBU) มาจำหน่าย
	
คำถาม	4) สถานีอัดประจุไฟฟ้าซึ่งเป็นปัจจัยที่มีส่วนในการเติบโตของรถยนต์ไฟฟ้าในประเทศมีความสอดคล้องกับการขยายตัวของความต้องใช้รถไฟฟ้าหรือไม่

สมมติฐาน สถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้ามีจำนวนเพิ่มขึ้นตามความต้องการใช้รถยนต์ไฟฟ้าที่เพิ่มสูงขึ้นของผู้บริโภค

คำตอบ 1. จากข้อมูลของ กฟน.พบว่าจำนวนผู้ใช้ไฟฟ้าที่เป็นสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้า (ตามข้อมูลที่ขึ้นทะเบียนกับ กฟน.) 
มีอัตราเพิ่มสูงขึ้นอย่างมากในช่วงปี 2 ปีที่ผ่านมา คือปี พ.ศ. 2564-2565โดยเฉพาะอย่างยิ่งในปี 2565 ที่เติบโตถึง 35% จากปี 2564 
ซึ่งจากกราฟจะเห็นได้ว่าจำนวนสถานีอัดประจุไฟฟ้าเพิ่มขึ้นอย่างรวดเร็วในปี 2565 ตั้งแต่ช่วงไตรมาส 2 เป็นต้นไป 
สอดคล้องนโยบายของภาครัฐที่มีออกมาเพื่อสนับสนุนการใช้และผลิตรถยนต์ไฟฟ้าในประเทศไทย

2. ในส่วนของสถานีอัดประจุไฟฟ้าทั่วประเทศนั้น พบว่าสถานีจะกระจุกตัวอยู่ตามจังหวัดที่มีความสำคัญทางเศรษฐกิจ
ซึ่งพบว่าจำนวนสถานีอัดประจุไฟฟ้าต่อจังหวัดส่วนใหญ่น้อยกว่า 20 สถานี โดยเฉลี่ยอยู่ที่ 9 สถานีต่อจังหวัดเท่านั้น
โดยพบว่ามี 4 จังหวัดที่ยังไม่มีสถานีอัดประจุไฟฟ้าไปติดตั้ง คือ กาฬสิน สมุทรสาคร อุทัยธานี และ ระนอง 
และยังก็พบว่ามีเพียง 6 จังหวัดที่มีสถานีอัดประจุมากกว่า 20 สถานี ได้แก่ กรุงเทพมหานคร นนทบุรี นครราชสีมา สมุทรปราการ ชลบุรี และเชียงใหม่ ตามลำดับ




	

```
df_plot3 = df_plot1
df_plot3['Total Non-Comercial'] = df_plot3.apply(lambda x: x['Passenger Car_Sub Total'] + x['Pickup 1 Ton_Sub Total'] , axis=1)
df_plot3
```
```
plt.figure(figsize=(15,5), dpi=150)

# (Optional) Set the format of datetime displayed in x-axis
ax = plt.gca()
formatter = mpl.dates.DateFormatter('%Y')
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y'))



ax.set_xlim(dt.datetime(2012, 1, 1), dt.datetime(2023, 1, 1))
# xlabel
# major tic


# Alternative 1: Plot with the given x and y
plt.plot(df_plot1['Year-Month'], df_plot3['Total Non-Comercial'],    # x and y to plot
         color='midnightblue', marker='o', linestyle='solid')    # The matplotlib linestyle )
ax.set_title('Non-commercial car production history past 10 Yrs')
ax.annotate('The first-car tax scheme End', xy=(pd.Timestamp('2013-01-01'), 220000),
            xytext=(pd.Timestamp('2012-05-01'), 100000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
            
ax.annotate('Covid-19:Lock down', xy=(pd.Timestamp('2020-04-01'), 24000),
            xytext=(pd.Timestamp('2018-01-01'), 23000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
ax.annotate('2014 Thai coup détat', xy=(pd.Timestamp('2014-06-01'), 145000),
            xytext=(pd.Timestamp('2014-02-01'), 60000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )
ax.annotate('Covid-19:Lock down', xy=(pd.Timestamp('2020-04-01'), 24000),
            xytext=(pd.Timestamp('2018-01-01'), 23000),
            bbox=dict(boxstyle='round', alpha=0.2),
            arrowprops=dict( arrowstyle='wedge,tail_width=0.5',alpha=0.1) )     
```
   ![image](https://user-images.githubusercontent.com/114766023/226256039-244994bf-ddc7-49a8-bc2c-8334bb2f4962.png)

   ![image](https://user-images.githubusercontent.com/114766023/226210841-276c357a-2df6-402f-b173-95055f136a28.png)


 
   ![image](https://user-images.githubusercontent.com/114766023/226211114-d01f6d0c-f930-4789-ac66-14966ba3ee46.png)

   ![image](https://user-images.githubusercontent.com/114766023/226212639-69a0af9d-cf11-47bf-b9ea-62ebbed849be.png)

```

```
 จากแผนที่จะเห็นว่ามีจุดของสถานีอัดประจุอยู๋เกือบทุกจังหวัด แต่ส่วนมากจะกระจุกตัวอยู๋ตามจังหวัดที่มีความสำคัญทางเศรษฐกิจ ซึ่งคาดว่าจะเป็น กรุงเทพมหานคร และปริมณฑล ชลบุรี ระยอง นครราชสีมา เชียงใหม่
เพื่อให้เห็นภาพชัดยิ่งขึ้นว่าจริงหรือไม่ จึงพลอตจำนวนสถานีอัดประจุไฟฟ้าเป็น Chroropleth ลงบนแผนที่เพื่อให้เห็นภาพชัดขึ้นต่อไป
![image](https://user-images.githubusercontent.com/114766023/226212147-2435c9f9-dcdf-418a-9656-28f061fa4a49.png)

![image](https://user-images.githubusercontent.com/114766023/226211340-bd80a0f8-11dd-4fcb-9747-6688b658e411.png)
![image](https://user-images.githubusercontent.com/114766023/226211345-8abaf546-e5b8-479b-8a1f-4c1d9e542bcf.png)
จาก Chroropleth เห็นได้ชัดว่าสถานีอัดประจุไฟฟ้าอยู่หนาแน่นตามจังหวัดที่กล่าวมาจริง

![image](https://user-images.githubusercontent.com/114766023/226211350-cae12129-5469-4844-b329-92cbdcf322a8.png)
```
print( f"Average Charging station per Province: {gdf_merged3.loc[ :, 'Number_charging_station'].mean():.0f}") 
```
Average Charging station per Province: 9

จาก KDE plot จะเห็นว่าจำนวนสถานีอัดประจุไฟฟ้าต่อจัดหวัดส่วนใหญ่นั้นมีน้อยกว่า 20 สถานี โดยเฉลี่ยอยู่ที่ 9 สถานีต่อจังหวัด
![image](https://user-images.githubusercontent.com/114766023/226211351-d3a1e803-d579-468f-b02d-3af08a952494.png)

กราฟแท่งนี้แสดงให้เห็นจำนวนสถานีอัดประจุไฟฟ้าในแต่ละจังหวัดอย่างชัดเจน โดยพบว่ามี 4 จังหวัดที่ยังไม่มีสถานีอัดประจุไฟฟ้าไปติดตั้ง คือ กาฬสิน สมุทรสาคร อุทัยธานี และ ระนอง และยังก็พบว่ามีเพียง 6 จังหวัดที่มีสถานีอัดประจุมากกว่า 20 สถานีได้แก่ กรุงเทพมหานคร นนทบุรี นครราชสีมา สมุทรปราการ ชลบุรี และเชียงใหม่ ตามลำดับ ดังตารางและกราฟด้านล่าง

![image](https://user-images.githubusercontent.com/114766023/226212245-bb66fde0-6f97-4130-ae0b-ca987f8dc084.png)
![image](https://user-images.githubusercontent.com/114766023/226211370-14d9a961-f8de-4d21-a0d6-680de36b4ea5.png)

## 5 : Conclusion and Suggestion
ข้อสรุปและความคิดเห็น
จากข้อมูลที่นยำเสนอมาสรุปได้ว่าอุตสาหกรรมยนต์ไทยฟื้นตัวขึ้นเกือบจะเท่าก่อนการระบาดของโควิด-19 และแนวโน้มค่อนข้างสดใส นอกจากนี้กระแสการเปลี่ยนผ่านสู่ยุครถยนต์ไฟฟ้ายังเกิดขึ้นอย่างรวดเร็ว
ตามสัดส่วนและประมาณยอดจดทะเบียนที่เพิ่มขึ้นอันเนื่องมาจากการส่งเสริมของรัฐ แต่สาธารณูปโถคสำคัญคือสถานีอัดประจุไฟฟ้ายังมีน้อยและกระจุกตามเมืองใหญ่ซึ่งทั้งภาครัฐและเอกชนต้องเร่งติดตั้ง
สถานีให้มากขึ้นทั่วประเทศเพื่อให้ผู้บริโภคเชื่อมั่นว่าสามารถใช้รถไฟฟ้าเดินทางข้ามต่างจังหวัดได้อย่างสบายใจ และตัดสินใจซื้อรถไฟฟ้าง่ายขึ้น
ข้อเสอแนะ
1.ความยากในการหา dataset และ จะเลือก dataset ไหนมาทำเลือกยากเพราะเวลามีจำกัด
2.การทำ Data Cleansing ทำได้ยากเนื่องจากข้อมูลอยู่ในรูป excel ซึ่งไม่เอื้อให้นำมาวิเคราะห์ได้่เลย ทำให้เสียเวลาส่วนนี้พอสมควร
3.การ Plot กราฟบางอันต้องอาศัยความเข้าใจ Libary และแต่ละ feature เพื่อให้กราฟอย่างที่ต้องการ
4.การ Plot map ด้วย GeoPandas เป็นเรื่องใหม่ ต้องทำความเข้าใจ Library พอสมควร รวมทั้งคุณสมบัตรของ Shape file อยากให้อาจารย์สอน libary 
นี้ด้วยเพราะมีประโยชน์ในการไปใช้งาน
