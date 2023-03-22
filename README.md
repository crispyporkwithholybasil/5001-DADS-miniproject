# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย

สืบเนื่องจากวิกฤตการระบาดของโรคโควิด-19 ในปี พ.ศ. 2563 ถึง 2564 (ค.ศ. 2020-21) ทำให้อุตสาหกรรมหลายอุตสาหกรรมได้รับผลกระทบอย่างมาก
จากการปิดเมือง (Lockdown) เพื่อควบคุมการระบาดทั่วโลก
อุตสาหกรรมยานยนต์เป็นหนึ่งในอุตสาหกรรมหลักในประเทศไทยก็ได้รับผลกระทบอย่างมากจากวิกฤตนี้เช่นกัน
ทางกลุ่มจึงมีความสนใจที่จะวิเคราะห์ดูแนวโน้มของอุตสาหกรรมจากช่วงก่อนและหลังวิกฤตโควิด-19 รวมทั้งกระแสของรถยนต์ไฟฟ้าที่เริ่มได้รับความสนใจจากผู้บริโภคในประเทศอันเนื่องมาจากมาตรการสนับสนุนการใช้รถไฟฟ้าของภาครัฐที่เริ่มมีผลในปีที่ผ่านมา(พ.ศ.2565)
ในการศึกษาครั้งนี้จะสนใจเฉพาะรถยนต์ที่ไม่ใช่เพื่อการพาณิชย์ได้แก่รถยนต์นั่งส่วนบุคคล (Passenger car) รถกระบะขนาด 1 ตัน (Pickup 1 ton truck) และรถกลุ่ม PPV เท่านั้น เพื่อสะท้อนความต้องการ(demand) ของประชาชนทั่วไป


## 1 : Datset
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
    
Source :  https://data.thaiauto.or.th
  - ข้อมูลการผลิตรถยนต์ภายในประเทศไทย
  - ข้อมูลผลิตรถยนต์เพื่อการส่งออกต่างประเทศ
  - สถิติรถจดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
  
1.2) สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้า ถึง ธ.ค. 2565 การไฟฟ้านครหลวง
     
Source : https://data.go.th/dataset/mea-customer
		 
1.3) ข้อมูล DC EV Charging Station Thailand - Piyamate Wisanuvej.kml
    
Source : http://bit.ly/401dOwX

1.4) ข้อมูลภูมิศาสตร์ระบุขอบเขตจังหวัดของประเทศไทย Thailand - Subnational Administrative Boundaries Dataframe 
     
Source : https://data.humdata.org/dataset/cod-ab-tha

## 2 : Data Cleansing and EDA

2.1) Install and Import Library

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

2.2.1) ข้อมูลการผลิตรถยนต์ภายในประเทศไทยและข้อมูลผลิตรถยนต์เพื่อการส่งออกรถยนต์ต่างประเทศ
```
df_prod_dom = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-production-thai.xlsx?raw=true",header=[0,1]) 
RangeIndex: 133 entries, 0 to 132
Data columns (total 19 columns):

df_export_unit = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-export-unit.xlsx?raw=true" ,header=[0,1]) 
RangeIndex: 132 entries, 0 to 131
Data columns (total 47 columns):
```
![image](https://user-images.githubusercontent.com/114766023/226757030-b429c9c9-f8d9-40ed-b3ac-2245d47813a0.png)

ลักษณะของข้อมูลที่พบและต้องจัดการคือ

1. ส่วนของ dataframe header เป็น 2 ชั้น จึงต้องทำการรวบให้เหลือเพียง 1 ชั้น

2. ชื่อเดือนมี white space ปนอยู่ และตัวเลขใน dataframe มีจุลภาค(comma)ต้องกำจัดออกก่อนจะแปลงเป็น data type ที่ต้องการ

3. ชื่อเดือนเป็นภาษาไทยจึงต้องทำการเปลี่ยนเป็นตัวเลขเพื่อให้สามารถแปลงให้เป็น Datetime format (dtype) เพื่อให้สามารถนำไปใช้งานต่อได้
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

|index|Year|Month|Year-Month|Passenger Car\_&lt;=1,500 CC|Passenger Car\_&gt;1,500 CC|Passenger Car\_Sub Total|Pickup 1 Ton\_Single Cab|Pickup 1 Ton\_Double Cab|Pickup 1 Ton\_PPV|Pickup 1 Ton\_Sub Total|
|---|---|---|---|---|---|---|---|---|---|---|
|0|2012|1|2012-01-01 00:00:00|26770|13508|40278|34498|52510|10167|97175|
|1|2012|2|2012-02-01 00:00:00|31309|15219|46528|40410|65555|11633|117598|
|2|2012|3|2012-03-01 00:00:00|35848|16930|53619|46375|71301|15636|133312|
|3|2012|4|2012-04-01 00:00:00|38554|13130|52425|29898|50587|9466|89951|
|4|2012|5|2012-05-01 00:00:00|57004|23626|81639|41174|70057|11584|122815|

|index|Year|Month|Year-Month|Asia\_Passenger Car|Asia\_Pick up|Asia\_PPV|Asia\_Other|Asia\_Total|Australia and Oceania\_Passenger Car|Australia and Oceania\_Pick up|Australia and Oceania\_PPV|Australia and Oceania\_Other|Australia and Oceania\_Total|Middle East\_Passenger Car|Middle East\_Pick up|Middle East\_PPV|Middle East\_Other|Middle East\_Total|Africa\_Passenger Car|Africa\_Pick up|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|2012|1|2012-01-01 00:00:00|12409|4487|2387|0|19283|1094|0|315|0|0|0|15808|50|0|15858|9|1841|
|1|2012|2|2012-02-01 00:00:00|5135|336|0|0|5471|150|949|0|0|1099|0|5461|0|0|5461|0|401|
|2|2012|3|2012-03-01 00:00:00|12989|9041|5723|0|27753|1716|15676|342|0|17734|495|23330|448|0|24273|19|2855|
|3|2012|4|2012-04-01 00:00:00|6172|6186|2607|0|14965|1078|9968|396|0|11442|759|14329|147|0|15235|12|2304|
|4|2012|5|2012-05-01 00:00:00|14363|8492|3668|0|26523|5255|14600|194|0|20049|671|21421|163|0|22255|267|2313|

2.2.2) ปริมาณรถยนต์จดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
ข้อมูลมีคุณภาพดี ไม่ต้องทำ Data Cleansing ใดๆ

```
new_register = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/New_Car_with_Fuel.csv?raw=true")
```
|index|Year|Month|Type|ICEV|HEV|PHEV|HEV &amp; PHEV|BEV|Total xEV|Not Specific|Other|Total|
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|2561|01|Bus|939|0|0|0|0|0|0|0|939|
|1|2561|01|Mini MPV|0|0|0|0|0|0|0|0|0|
|2|2561|01|Passenger Car and Pickup Truck|89115|0|0|1049|0|1049|2|0|89117|
|3|2561|01|Three Wheeler|14|0|0|0|0|0|0|0|14|
|4|2561|01|Truck|4227|0|0|0|0|0|0|1504|5731|

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 335 entries, 0 to 334
Data columns (total 12 columns):
 #   Column        Non-Null Count  Dtype 
---  ------        --------------  ----- 
 0   Year          335 non-null    int64 
 1   Month         335 non-null    object
 2   Type          335 non-null    object
 3   ICEV          335 non-null    int64 
 4   HEV           335 non-null    int64 
 5   PHEV          335 non-null    int64 
 6   HEV & PHEV    335 non-null    int64 
 7   BEV           335 non-null    int64 
 8   Total xEV     335 non-null    int64 
 9   Not Specific  335 non-null    int64 
 10  Other         335 non-null    int64 
 11  Total         335 non-null    int64 
dtypes: int64(10), object(2)
memory usage: 31.5+ KB
```

2.2.3) สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้า ถึง ธ.ค. 2565 การไฟฟ้านครหลวง

![image](https://user-images.githubusercontent.com/114766023/226974536-990169c3-7c73-47de-8119-65812dbf013c.png)

ลักษณะของข้อมูลที่พบและต้องจัดการคือ
1. ข้อมูลมีแถวที่เป็น NaN ทั้งส่วนบนและส่วนล่างของ dataframe เนื่องจากข้อมูลเก็บมาในรูปแบบ Excel จึงต้องตัดออกและเลือกเฉพาะคอลัมน์ 'สถานีอัดประจุไฟฟ้า' มาใช้ในการวิเคราะห์

2. ข้อมูลที่ได้รับมาอยู่ในรูปปี พุทธศักราช(B.E.) ส่งผลให้ไม่สามารถแปลงข้อมูลให้อยู่ในรูปของ datetime format ได้ จึงต้องทำการแปลงให้เป็นคริสต์ศักราช (A.D.)
```
charger_df = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EV_Charger(x-).csv?raw=true",header = 2)
charger_df.info()
```
|index|ปี|เดือน|Unnamed: 2|บ้านอยู่อาศัย|กิจการ    ขนาดเล็ก|กิจการ     ขนาดกลาง|กิจการ      ขนาดใหญ่|กิจการ     เฉพาะอย่าง|องค์กรที่ไม่แสวงหากำไร|สูบน้ำเพื่อการเกษตร|Unnamed: 10|สถานีอัดประจุไฟฟ้า|    จำนวนผู้ใช้ไฟฟ้า     |Unnamed: 13|Unnamed: 14|Unnamed: 15|Unnamed: 16|Unnamed: 17|Unnamed: 18|Unnamed: 19|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|NaN|NaN|NaN|NaN|NaN|NaN|NaN|NaN|NaN|NaN|ผู้ใช้ไฟฟ้าชั่วคราว|สำหรับยานยนต์ไฟฟ้า \*|NaN|NaN|NaN|NaN|NaN|NaN|NaN|NaN|
|1|NaN|NaN|NaN|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|\(ราย\)|NaN|NaN|NaN|NaN|NaN|NaN|NaN|
|2|2562\.0|ม\.ค\.|NaN|3,250,102|510,189|23,355|2,392|3,202|318| -   |26,286|93|3,815,937|NaN|NaN|NaN|NaN|NaN|NaN|NaN|
|3|2562\.0|ก\.พ\.|NaN|3,259,884|510,869|23,429|2,398|3,209|318| -   |26,490|101|3,826,698|NaN|NaN|NaN|NaN|NaN|NaN|NaN|
|4|2562\.0|มี\.ค\.|NaN|3,269,233|511,305|23,488|2,396|3,230|324| -   |26,640|106|3,836,722|NaN|NaN|NaN|NaN|NaN|NaN|NaN|

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
เปลี่ยนเลขปีเป็น ค.ศ.และแปลงเป็น datetime format (dtype)
```
charger_df7 = charger_df6.rename(columns={'ปี': 'Year','เดือน':'Month'})
charger_df7['Year'] = charger_df7['Year']-543
charger_df7.insert(2, 'Year-Month', pd.to_datetime(charger_df7[['Year', 'Month']].assign(Day=1)) )
charger_df7
```
|index|Year|Month|Year-Month|สถานีอัดประจุไฟฟ้า|
|---|---|---|---|---|
|2|2019|1|2019-01-01 00:00:00|93|
|3|2019|2|2019-02-01 00:00:00|101|
|4|2019|3|2019-03-01 00:00:00|106|
|5|2019|4|2019-04-01 00:00:00|111|
|6|2019|5|2019-05-01 00:00:00|115|

2.2.4) ข้อมูลภูมิศาสตร์ระบุขอบเขตจังหวัดของประเทศไทยและ DC EV Charging Station Thailand

```
#Import Thailand map data
#Thailand - Subnational Administrative Boundaries Dataframe from https://data.humdata.org/dataset/cod-ab-tha
gdf = gpd.read_file('tha_admbnda_adm1_rtsd_20190221.shp')
gdf.head()
```

![image](https://user-images.githubusercontent.com/114766023/226981497-04ba409a-6cb9-4437-ad4c-fd9698cafadf.png)

```
#Read all data chager locations of each brand.
df_ea24 = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EA_24hr.csv?raw=true'
df_eaop= pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EA_Off_peak.csv?raw=true')
df_elex = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EleX.csv?raw=true')
df_evolt = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EVolt_Sharge_etc.csv?raw=true')
df_mg = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/MG.csv?raw=true')
df_pea = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/PEA.csv?raw=true')
df_ptt = pd.read_csv('https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/PTT.csv?raw=true')
df_charger = pd.concat([df_ea24,df_eaop,df_elex,df_evolt,df_mg,df_pea,df_ptt])
```
![image](https://user-images.githubusercontent.com/114766023/226987236-18d6ccd8-520a-4117-9768-e0d65a6396ab.png)

ลักษณะของข้อมูลที่พบและต้องจัดการคือ
1. เนื่องจากไฟล์ DC EV Charging Station Thailand - Piyamate Wisanuvej เป็น .kml file จึงนำไปแปลงเป็น .csv file ด้วยเว็ป https://mygeodata.cloud/converter/ ก่อน ซึ่งได้ออกมาทั้งหมด 7 ไฟล์ตามประเภทของผู้ให้บริการ แล้วจึงต้องทำการรวมไฟล์ทั้ง 7 ไฟล์เป็น dataframe เดียว

2. DC EV Charging Station Thailand ข้อมูลที่ตั้งเป็นภาษาไทยและยากที่จะแยกเอาจังหวัดออกมา จึงใช้วิธีการนำข้อมูล Latitude และ Longitude ไปเทียบกับ Polygon geometry ในGeoDataframe ประเทศไทย เพื่อเอาชื่อจังหวัดจากแผนที่มาเพิ่มข้อมูลของจังหวัดของที่ตั้งสถานีก่อนนำมาวิเคราะห์

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
								     
![image](https://user-images.githubusercontent.com/114766023/226986963-29716a33-8995-48a4-9a94-4b990a104213.png)

## 3 : EDA
ในการศึกษาครั้งนี้จะสนใจเฉพาะรถยนต์ที่ไม่ใช่เพื่อการพาณิชย์ได้แก่รถยนต์นั่งส่วนบุคคล (Passenger car) รถกระบะขนาด 1 ตัน (Pickup 1 ton truck) และรถกลุ่ม PPV เท่านั้น


3.1) แนวโน้มปริมาณการผลิตยานยนต์สำหรับในประเทศไทย
ในที่นี้ใช้ข้อมูลยอดการผลิตสำหรับในประเทศแทนทิศทางของอุตสาหกรรมรถยนต์ภายในประเทศเนื่องจากหากยอดสั่งซื้อ/ผลิตสูงขึ้นหรือลดลง อุตสาหกรรมและธุรกิจที่เกี่ยวข้องในประเทศก็จะได้รับผลกระทบและไปในทิศทางเดียวกัน
ดูแนวโน้มการเปลี่ยนแปลงของรถยนต์ที่ผลิตเพื่อจำหน่ายในประทศไทยเป็นรายปีและสัดส่วนของรถยนต์นั่งส่วนบุคคลกับรถกระบะขนาด 1 ตันในแต่ละปี

![image](https://user-images.githubusercontent.com/114766023/226995188-afc82d87-396f-4c3b-a5a7-5ddb9f6708dd.png)

![image](https://user-images.githubusercontent.com/114766023/226315791-6326c04e-1391-444a-8b77-da0c70d96ea6.png)

เพื่อจะดูแนวโน้มของของ Non-commercial car ในแต่ละเดือนจึงรวมปริมาณของ Passenger car กับ Pickup 1 Ton ก่อน แล้วจึงพลอต Line chart และเพิ่มเหตุการณ์สำคัญที่น่าจะมีผลต่ออุตสากรรมเข้าไปในกราฟ

![image](https://user-images.githubusercontent.com/114766023/226317965-781dc0bd-a93e-49c0-8f9e-12296d83fa7a.png)

ปรับช่วงข้อมูลให้มาอยู่ในช่วง 5 ปีย้อนหลัง เพื่อให้เห็นแนวโน้มชัดขึ้น
![image](https://user-images.githubusercontent.com/114766023/226318066-9709ee02-6552-42e9-86a0-30344d163ab7.png)

โดยจากข้อมูลและกราฟสรุปได้ว่า
1. อุตสาหกรรมรถยนต์ในประเทศมีการฟื้นตัวหลังจากวิกฤตการระบาดของโรคโควิด-19 แต่ยังไม่เท่ากับช่วงปี 2020 อาจจะด้วยปัจจัยจากสภาพเศรษฐกิจที่อยู่ในภาวะเงินเฟ้อซึ่งยังไม่สามารถระบุได้ชัดเจนด้วยข้อมูลที่มี
2. สัดส่วนการผลิต/ซื้อรถกระบะมีมากกว่ารถยนต์ส่วนบุคคล(รถเก๋ง)และสัดส่่วนของรถกระบะยังเพิ่มขึ้นอย่างต่อเนื่อง

3.2) ข้อมูลผลิตรถยนต์เพื่อการส่งออกต่างประเทศ

ในที่นี้ใช้ข้อมูลยอดการผลิตเพื่อการส่งออกไปต่างประเทศของอุตสาหกรรมรถยนต์ไทยแทนมุมมองของภาวะตลาดรถยนต์โลก

![image](https://user-images.githubusercontent.com/114766023/226757738-e20a3cf0-54d3-416b-9736-933c00e5bc87.png)

เนื่องจากกราฟทับซ้อนกันมากจนทำให้ดูแนวโน้มได้ยากจึงแยกกราฟออกเป็น subplot ตาม Region ทั้ง 8 ดังนี้



![image](https://user-images.githubusercontent.com/114766023/226348572-163556d8-623e-4b87-84c5-5d27e66a8218.png)

จากกราฟทำให้ทราบว่า
1. ไทยส่งออกรถยนต์ไปตลาดเอเชียนเป็นอันดับ1 และออสเตรเลียเป็นอันดับ2
2. หลังจากวิกฤตการระบาดของโรคโควิด-19(ปี 2020-21) พบว่ายอดการผลิตเพื่อส่งออกสู่ทุกภูมิภาคกลับมาเป็นขาขึ้นโดยเฉพาะอย่างยิ่งตลาดเอเชียและออสเตรเลียที่เพิ่มสูงขึ้นจนมากกว่าก่อนช่วงวิกฤตโควิด-19 แล้ว แต่บางตลาดยังฟื้นไม่มากอาจจะมาจากภาวะเงินเฟ้อที่เกิดขึ้นในปี 2022 ซึ่งยังไม่ได้สามารถสรุปได้ด้วยข้อมุลที่มีในที่นี้

3.3) ปริมาณรถยนต์จดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
ดูแนวโน้มของรถยนต์จดทะเบียนใหม่ย้อนหลัง 5 ปี และดูสัดส่วนของรถยนต์แต่ละประเภทเชื้อเพลิงในแต่ละปีด้วย Stack bar chart
(ICE =  internal combustion engine, PHEV = Plug-In Hybrid Electic vehicle, HEV = Hybrid Electric Vehicle, BEV = Baterry Electric Vehicle)
ซึ่งพบว่ากราฟปี 2565 มีปริมาณเพิ่มขึ้นจากปี 2564 ซึ่งพ้นจากภาวะการระบาดของโควิด-19

![image](https://user-images.githubusercontent.com/114766023/226210712-13c23bec-82a0-4f45-96fd-510e5a7043bc.png)

เลือกดูในส่วนเฉพาะของ EV ว่ามีสัดส่วนเป็นอย่างไรในช่วง 3 ปีที่ผ่านมา นับตั้งแต่เกิดการระบาดของโควิด-19

![image](https://user-images.githubusercontent.com/114766023/226322538-15dd7df8-f317-43e4-b20d-5486f7739028.png)

จากกราฟพบว่าสัดส่วนของรถยนต์ไฟฟ้า(PHEV,HEV,BEV) รวมกันต่อสัดส่วนรถยนต์ประเภทเครื่องยนต์สันดาปภายในเพิ่มขึ้นทุกปีรวมทั้งปริมาณรถยนต์ใหม่ก็เพิ่มขึ้นมากด้วยเช่นกัน เห็นได้ชัดจากยอดจดทะเบียนปี พ.ศ.2564 ประมาณ 43,000 คัน เพิ่มขึ้นมาเป็นเกือบ 83,000 คัน ในปี พ.ศ.2565 หรือเกือบเท่าตัวเลยทีเดียว

|Year|HEV|PHEV|BEV|
|---|---|---|---|
|2563|12995|1084|1288|
|2564|34339|7060|1958|
|2565|62137|11117|9454|

เทียบสัดส่วนปีต่อปี 2564-2565 YoY
![image](https://user-images.githubusercontent.com/114766023/226323639-6300e468-d4e2-4597-89a1-f5ba846f2851.png)

เปรียบเทียบปริมาณรถ EV ที่จดทะเบียนใหม่แต่ละประเภท และดูอัตราการเติบโตเทียบกับปีก่อนหน้า (%YOY Growth rate)
![image](https://user-images.githubusercontent.com/114766023/226210815-94db3279-77a3-4fe5-855e-0e62673ff223.png)
![image](https://user-images.githubusercontent.com/114766023/226324759-7059ddf1-e77b-4af6-b5a1-8e02a6e9522d.png)

จากสัดส่วนรถยนต์ไฟฟ้า(EV) แต่ละประเภทพบว่าสัดส่วนการเติบโตของรถยนต์ Battery EV (BEV) มีเพิ่มมากขึ้นเมื่อเทียบกับ Hybrid EV (HEV) และ Plug-In Hybrid EV (PHEV) โดยสัดส่วน BEV เทียบกับ EV อื่นๆ เพิ่มขึ้นจาก 4.5% ในปี 2564 เป็น 11.4% ในปี 2565 และยอดจดทะเบียนปี 2565 เพิ่มขึ้นจาก ปี 2564 คิดเป็นอัตราการเติบโตสูงถึงกว่า 79%

ในขณะที่ PHEV กับ HEV ถึงจะมียอดเพิ่มขึ้นแต่เทียบเป็นอัตราการเติบโตแล้วกลับพบว่าลดลงสวนทางกับ BEV ซึ่งอาจจะกล่าวได้ว่าประเทศไทยกำลังเปลี่ยนผ่านสู่ยุครถยนต์ไฟฟ้าแบบใช้ไฟฟ้า 100% เช่น BEV และสัดส่วนของรถไฟฟ้ามีแนวโน้มจะกินส่วนแบ่งรถยนต์สันดาปภายใน (Internal Combustion Engine - ICE) เพิ่มขึ้นทุกปีตามมาตราการที่รัฐส่งเสริมทั้งฝั่งผู้บริโภคคือการลดภาษีและฝั่งผู้ผลิตที่ต้องผลิตรถไฟฟ้าให้มากกว่ายอดที่นำเข้าทั้งคัน (CBU) มาจำหน่าย

![image](https://user-images.githubusercontent.com/114766023/226993286-67206ba9-b1b3-469e-ab79-bf5f65a80133.png)

**Source : https://www.bangkokbiznews.com/business/988550**

3.4) สถิติจำนวนผู้ใช้ไฟฟ้า แยกตามประเภทผู้ใช้ไฟฟ้าถึง ธ.ค. 2565 การไฟฟ้านครหลวง ใช้ในการหาจำนวนสถานีอัดประจุในพื้นที่ กทม.

คำนวนหาอัตราการเพิ่มขึ้นของสถานีอัดประจุจากปีก่อนหน้า (%YOY Growth rate)
```
diff = (charger_df4['สถานีอัดประจุไฟฟ้า'] - charger_df4['สถานีอัดประจุไฟฟ้า'].shift(1))/charger_df4['สถานีอัดประจุไฟฟ้า']

charger_df4['percent_growth'] = diff*100
charger_df4
```
![image](https://user-images.githubusercontent.com/114766023/226256039-244994bf-ddc7-49a8-bc2c-8334bb2f4962.png)

จากกราฟแสดงให้เห็นว่าในช่วง 3 ปีย้อนหลังมานี้ จำนวนสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้าในกรุงเทพฯ ที่จดทะเบียนกับ กฟน. มีอัตราเพิ่มขึ้นทุกปีและโดยเฉพาะอย่างในปี 2565 ที่มีอัตราการ เติบโตจากปี 2564 ถึง 35% YoY

![image](https://user-images.githubusercontent.com/114766023/226325682-dc549937-4b99-4ec5-8314-9dd8976db1eb.png)

จากกราฟจะเห็นได้ว่าจำนวนสถานีอัดประจุไฟฟ้าเพิ่มขึ้นอย่างรวดเร็วในปี 2565 ตั้งแต่ช่วงไตรมาส 2 เป็นต้นไป สอดคล้องนโยบายของภาครัฐที่มีออกมาเพื่อสนับสนุนการใช้และผลิตรถยนต์ไฟฟ้าในประเทศไทย

3.5) ข้อมูลจำนวนสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้าประเภท DC ในประเทศไทย

```
df_charger_provider = df_charger.pivot_table(index='Provider',values='Name',aggfunc='count').reset_index()
df_charger_provider = df_charger_provider.rename({'Name':'Number_of_station'}, axis='columns')
df_charger_provider
```

|index|Provider|Number\_of\_station|
|---|---|---|
|0|Altervim|4|
|1|EA|216|
|2|EGAT|4|
|3|EVolt|10|
|4|EleX|64|
|5|GWM|5|
|6|MG|128|
|7|PEA|131|
|8|PTT|150|
|9|Sharge|9|
|10|Tesla|1|

จากแผนที่จะเห็นว่ามีจุดของสถานีอัดประจุอยู่เกือบทุกจังหวัด แต่ส่วนมากจะกระจุกตัวอยู่ตามจังหวัดที่มีความสำคัญทางเศรษฐกิจ ซึ่งคาดว่าจะเป็น กรุงเทพมหานคร และปริมณฑล ชลบุรี ระยอง นครราชสีมา เชียงใหม่

![image](https://user-images.githubusercontent.com/114766023/226212147-2435c9f9-dcdf-418a-9656-28f061fa4a49.png)

เพื่อให้เห็นภาพชัดยิ่งขึ้นว่าจริงหรือไม่ จึงพลอตจำนวนสถานีอัดประจุไฟฟ้าเป็น Choropleth ลงบนแผนที่เพื่อให้เห็นภาพชัดขึ้นต่อไป

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

กราฟแท่งนี้แสดงให้เห็นจำนวนสถานีอัดประจุไฟฟ้าในแต่ละจังหวัดอย่างชัดเจน โดยพบว่ามี 4 จังหวัดที่ยังไม่มีสถานีอัดประจุไฟฟ้าไปติดตั้ง คือ กาฬสิน สมุทรสงคราม อุทัยธานี และ ระนอง และยังก็พบว่ามีเพียง 6 จังหวัดที่มีสถานีอัดประจุมากกว่า 20 สถานีได้แก่ กรุงเทพมหานคร นนทบุรี นครราชสีมา สมุทรปราการ ชลบุรี และเชียงใหม่ ตามลำดับ ดังตารางและกราฟด้านล่าง

|index|Province|Number\_charging\_station|
|---|---|---|
|0|Bangkok|176|
|1|Nonthaburi|47|
|2|Nakhon Ratchasima|39|
|3|Samut Prakan|31|
|4|Chon Buri|28|
|5|Chiang Mai|24|

![image](https://user-images.githubusercontent.com/114766023/226211370-14d9a961-f8de-4d21-a0d6-680de36b4ea5.png)

## 4 : Question and Answer

คำถาม: 1) สถานการณ์การผลิตรถยนต์ส่งออกทั่วโลกจากอดีตถึงปัจจุบันเป็นอย่างไร

สมมติฐาน:
สถานการณ์การผลิตรถยนต์ส่งออกทั่วโลกมีการฟื้นตัวขึ้นหลังภาวะวิกฤตโควิด-19

คำตอบ: 
จากกราฟและข้อมูล สรุปได้ว่าการสถานการณ์การผลิตรถยนต์ส่งออกทั่วโลกที่ได้รับผลกระทบจากวิกฤตการระบาดของโรคโควิด-19 ฟื้นตัวกลับมาจนเข้าสู่ภาวะเหมือนก่อนการระบาดแล้ว

คำถาม: 2) การผลิตรถยนต์เพื่อจำหน่ายในประเทศไทยยังสดใสหรือไม่

สมมติฐาน:
การผลิตรถยนต์เพื่อจำหน่ายในประเทศไทยฟื้นตัวและมีแนวโน้มเติบโตขึ้นหลังจากผ่านวิกฤตโควิด-19

คำตอบ: 
ในที่นี้ใช้ข้อมูลยอดการผลิตสำหรับในประเทศแทนทิศทางของอุตสาหกรรมรถยนต์ภายในประเทศเนื่องจากหากยอดสั่งซื้อ/ผลิตสูงขึ้นหรือลดลง อุตสาหกรรมและธุรกิจที่เกี่ยวข้องในประเทศก็จะได้รับผลกระทบและไปในทิศทางเดียวกัน โดยจากข้อมูลและกราฟสรุปได้ว่า

1. อุตสาหกรรมรถยนต์ในประเทศมีการฟื้นตัวหลังจากวิกฤตการระบาดของโรคโควิด-19 แต่ยังไม่เท่ากับช่วงปี 2020 อาจจะด้วยปัจจัยจากสภาพเศรษฐกิจที่อยู่ในภาวะเงินเฟ้อซึ่งยังไม่สามารถระบุได้ชัดเจนด้วยข้อมูลที่มี


2. สัดส่วนการผลิต/ซื้อรถกระบะมีมากกว่ารถยนต์นั่งส่วนบุคคล(รถเก๋ง)และสัดส่่วนของรถกระบะยังเพิ่มขึ้นอย่างต่อเนื่อง

คำถาม: 3) รถยนต์ไฟฟ้าในประเทศประเทศไทยทิศทางและการเติบโตเป็นอย่างไร

สมมติฐาน: จากนโบายของรัฐที่ออกมาส่งเสริมการใช้และผลิตรถไฟฟ้าในประเทศที่ออกมาปลายปี2565 
	ทำให้คาดว่ารถยนต์ไฟฟ้ามีการเติบโตของผู้ใช้ที่สูงขึ้นและมีทิศทางที่กินส่วนแบ่งของรถยนต์สันดาปภายในมากขึ้นเรื่อยๆ

คำตอบ: 	จากการวิเคราะห์ยอดจดทะเบียนรถยนต์ใหม่ตั้งแต่ปี 2018-2022 พบว่าเป็นไปตามสมมติฐาน โดยสัดส่วนของรถยนต์ไฟฟ้า(PHEV,HEV,BEV) รวมกันมีสัดส่วนเทียบกับรถยนต์สันดาปภายในขึ้นทุกปี 

และจากสัดส่วนรถยนต์ไฟฟ้า(EV) แต่ละประเภทพบว่าสัดส่วนการเติบโตของรถยนต์ Battery EV (BEV) มีเพิ่มมากขึ้นเมื่อเทียบกับ Hybrid EV (HEV) และ Plug-In Hybrid EV (PHEV) 

และสัดส่วนของรถไฟฟ้ามีแนวโน้มจะกินส่วนแบ่งรถยนต์แสันดาปภายใน (Internal Combustion Engine - ICE)  เพิ่มขึ้นทุกปี
ตามมาตราการที่รัฐส่งเสริมทั้งฝั่งผู้บริโดภคคือการลดภาษีและฝั่งผู้ผลิตที่ต้องผลิตรถไฟฟ้าให้มากกว่ายอดที่นำเข้าทั้งคัน (CBU) มาจำหน่าย
	
คำถาม :	4) สถานีอัดประจุไฟฟ้าซึ่งเป็นปัจจัยที่มีส่วนในการเติบโตของรถยนต์ไฟฟ้าในประเทศมีความสอดคล้องกับการขยายตัวของความต้องใช้รถไฟฟ้าหรือไม่
สมมติฐาน สถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้ามีจำนวนเพิ่มขึ้นตามความต้องการใช้รถยนต์ไฟฟ้าที่เพิ่มสูงขึ้นของผู้บริโภค

คำตอบ : 

จำนวนผู้ใช้ไฟฟ้าที่เป็นสถานีอัดประจุไฟฟ้าสำหรับรถยนต์ไฟฟ้าใน กรุงเทพมหานคร (ตามข้อมูลที่ขึ้นทะเบียนกับ กฟน.) มีอัตราเพิ่มสูงขึ้นอย่างมากในช่วงปี 2 ปีที่ผ่านมา โดยเพิ่มขึ้นอย่างรวดเร็วในปี 2565 ตั้งแต่ช่วงไตรมาส 2 เป็นต้นไป สอดคล้องนโยบายของภาครัฐที่มีออกมาเพื่อสนับสนุนการใช้และผลิตรถยนต์ไฟฟ้าในประเทศไทย

แต่ในส่วนของสถานีอัดประจุไฟฟ้าทั่วประเทศนั้น พบว่าสถานีจะกระจุกตัวอยู่ตามจังหวัดที่มีความสำคัญทางเศรษฐกิจและพบว่าจำนวนสถานีอัดประจุไฟฟ้าต่อจังหวัดส่วนใหญ่น้อยกว่า 20 สถานี โดยเฉลี่ยอยู่ที่ 9 สถานีต่อจังหวัดเท่านั้น โดยพบว่ามี 4 จังหวัดที่ยังไม่มีสถานีอัดประจุไฟฟ้าไปติดตั้ง

## 5 : Conclusion and Suggestion
ข้อสรุปและความคิดเห็น :

จากข้อมูลที่นำมาเสนอ สรุปได้ว่าอุตสาหกรรมยนต์ไทยฟื้นตัวขึ้นเกือบจะเท่าก่อนการระบาดของโควิด-19 และแนวโน้มค่อนข้างสดใส นอกจากนี้กระแสการเปลี่ยนผ่านสู่ยุครถยนต์ไฟฟ้ายังเกิดขึ้นอย่างรวดเร็ว ตามสัดส่วนและประมาณยอดจดทะเบียนที่เพิ่มขึ้น อันเนื่องมาจากการส่งเสริมของรัฐ แต่สาธารณูปโภคสำคัญคือ สถานีอัดประจุไฟฟ้ายังมีน้อยและกระจุกตามเมืองใหญ่ซึ่งทั้งภาครัฐและเอกชนจะต้องเร่งติดตั้งสถานีให้มากขึ้นทั่วประเทศ เพื่อให้ผู้บริโภคเชื่อมั่นว่าสามารถใช้รถไฟฟ้าเดินทางข้ามต่างจังหวัดได้อย่างสบายใจ และตัดสินใจซื้อรถไฟฟ้าได้ง่ายขึ้น

ข้อเสอแนะและอุปสรรคที่พบ :

1. ความยากในการหา dataset และ การเลือก dataset มาวิเคราะห์เพราะเวลามีจำกัด

2. การทำ Data Cleansing ทำได้ยาก เนื่องจากข้อมูลอยู่ในรูป excel ซึ่งไม่เอื้อให้นำมาวิเคราะห์ได้เลย ต้องทำให้เสียเวลาส่วนนี้พอสมควร

3. ข้อมูลการจดทะเบียนรถยนต์ใหม่ถ้าสามารถระบุลงไประดับรายจังหวัดได้จะทำให้เห็นภาพของการกระจายและทิศทางที่ลึกมากขึ้น

4. การพลอตกราฟบางอันต้องอาศัยความเข้าใจ Library และแต่ละ feature เพื่อให้กราฟอย่างที่ต้องการ


5. การพลอตแผนที่ด้วย GeoPandas เป็นเรื่องใหม่ ต้องทำความเข้าใจ Library พอสมควร รวมทั้งคุณสมบัตรของ Shape file อยากให้อาจารย์สอน library นี้ด้วยเพราะมีประโยชน์ในการไปใช้งาน

