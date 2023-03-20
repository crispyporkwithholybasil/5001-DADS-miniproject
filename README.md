# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย
## 1 : Datset
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
    
	Source :  https://data.thaiauto.or.th
  - ข้อมูลการผลิตรถยนต์ภายในประเทศไทย
  - ข้อมูลผลิตรถนยนต์เพื่อการส่งออกรถยนต์ต่างประเทศ
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



## 3 : Question and Answer
3.1) สถานการณ์การผลิตรถยนต์จากอดีตถึงปัจจุบันทั่วโลกเป็นอย่างไร

  ![image](https://user-images.githubusercontent.com/114766023/226210595-15079ccd-2325-4c31-b31a-ef57f9b1be1d.png)

เนื่องจากกราฟมีจำนวนหลายเส้นและตัดกันจึงทำการแยกกราฟของแต่ละภูมิภาคออกเป็น subplot เพื่อให้เห็นแนวโน้มที่ชัดเจนขึ้น

  ![image](https://user-images.githubusercontent.com/114766023/226210568-c4db94dc-d0f5-4178-857b-72a77bdc1f42.png)
จากกราฟพบว่า....

3.2) อุตสาหกรรมรถยนต์ในประเทศไทยยังสดใสหรือไม่
       สมมติฐาน :  อุตสาหกรรมผลิตรถยนต์ในประเทศไทยฟื้นตัวและมีแนวโน้มเติบโตขึ้นหลังจากผ่านวิกฤตโควิด19
   ![image](https://user-images.githubusercontent.com/114766023/226211004-6574fe8e-26cf-4e76-b9fb-4d4b5f302e9a.png)
   ![image](https://user-images.githubusercontent.com/83213407/226184043-4bdaade1-91af-4633-9d35-932ee2f0f0d3.png)
   
   ![image](https://user-images.githubusercontent.com/114766023/226211647-6f14dae4-9a84-4541-9338-bcb3e68b5b38.png)

3.3) รถยนต์ไฟฟ้าในประเทศประเทศไทยทิศทางและการเติบโตเป็นอย่างไร     
   ![image](https://user-images.githubusercontent.com/114766023/226210712-13c23bec-82a0-4f45-96fd-510e5a7043bc.png)

   ![image](https://user-images.githubusercontent.com/114766023/226210773-a5b1a634-bf9d-4912-abe6-bbe9dddf335c.png)
 	
   ![image](https://user-images.githubusercontent.com/114766023/226210815-94db3279-77a3-4fe5-855e-0e62673ff223.png)
  
   ![image](https://user-images.githubusercontent.com/114766023/226210826-9fc543ff-8c5b-494b-9993-763f5925f309.png)
 
   ![image](https://user-images.githubusercontent.com/114766023/226256039-244994bf-ddc7-49a8-bc2c-8334bb2f4962.png)

   ![image](https://user-images.githubusercontent.com/114766023/226210841-276c357a-2df6-402f-b173-95055f136a28.png)

3.4) สถานีอัดประจุไฟฟ้าซึ่งเป็นปัจจัยที่มีส่วนในการเติบโตของรถยนต์ไฟฟ้าในประเทศมีความสอดคลองกับการขยายตัวของความต้องใช้รถไฟฟ้าหรือไม่
 
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

## 4 : Summary and Suggestion
