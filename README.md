# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย
สำหรับ Mini-project นี้จะประกอบไปด้วยการนำเสนอ 4 ส่วนได้กัน ได้แก่

1.  Source of Information (ชุดข้อมูลที่กลุ่มเราเลือกใช้)
2.  Cleansing DATA 
3.  Question and Answer (EDA)
4.  Summary and Suggestion

## 1 : Data and Source
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
  - ข้อมูลการผลิตรถนยนต์ภายในประเทศไทย
  - ข้อมูลผลิตรถนยนต์เพื่อการส่งออกรถยนต์จากประเทศ
  - สถิติรถจดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง
  - 
1.2) ข้อมูลจาก



## 2 : DATA Cleansing 
2.1) Import Library & Import Data
```
#Install library
!pip install geopandas 
!pip install descartes 
!pip install contextily
!pip install mapclassify
```
```
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
```
#Import data
df_prod_dom = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-production-thai.xlsx?raw=true",header=[0,1]) 
df_prod_dom.info()
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 133 entries, 0 to 132
Data columns (total 19 columns):
 #   Column                              Non-Null Count  Dtype  
---  ------                              --------------  -----  
 0   (Unnamed: 0_level_0, ลำดับที่)      132 non-null    float64
 1   (Unnamed: 1_level_0, ปี)            132 non-null    float64
 2   (Unnamed: 2_level_0, เดือน/ไตรมาส)  133 non-null    object 
 3   (Passenger Car, <=1,500 CC)         133 non-null    object 
 4   (Passenger Car, >1,500 CC)          133 non-null    object 
 5   (Passenger Car, Sub Total)          133 non-null    object 
 6   (Pickup 1 Ton, Single Cab)          133 non-null    object 
 7   (Pickup 1 Ton, Double Cab)          133 non-null    object 
 8   (Pickup 1 Ton, PPV)                 133 non-null    object 
 9   (Pickup 1 Ton, Sub Total)           133 non-null    object 
 10  (Pickup 1 Ton, VAN)                 133 non-null    object 
 11  (Bus, <=10 Tons)                    133 non-null    int64  
 12  (Bus, >10 Tons)                     133 non-null    object 
 13  (Bus, Sub Total)                    133 non-null    object 
 14  (Truck, <=5 Tons)                   133 non-null    object 
 15  (Truck, 5-10 Tons)                  133 non-null    object 
 16  (Truck, >10 Tons)                   133 non-null    object 
 17  (Truck, Sub Total)                  133 non-null    object 
 18  (Truck, รวม (คัน))                  133 non-null    object 
dtypes: float64(2), int64(1), object(16)
memory usage: 19.9+ KB
```
```
df_export_unit = pd.read_excel("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/stat-auto-export-unit.xlsx?raw=true" ,header=[0,1]) 
df_export_unit.columns = df_export_unit.columns.map('_'.join)
df_export_unit = df_export_unit.rename(columns={'Unnamed: 1_level_0_ปี': 'Year','Unnamed: 2_level_0_เดือน/ไตรมาส': 'Month'})
df_export_unit = df_export_unit[:-1]
df_export_unit = df_export_unit.drop('Unnamed: 0_level_0_ลำดับที่', axis=1)
display(df_export_unit)
df_export_unit.info()
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 132 entries, 0 to 131
Data columns (total 47 columns):
 #   Column                                      Non-Null Count  Dtype  
---  ------                                      --------------  -----  
 0   Year                                        132 non-null    float64
 1   Month                                       132 non-null    object 
 2   Asia_Passenger Car                          132 non-null    object 
 3   Asia_Pick up                                132 non-null    object 
 4   Asia_PPV                                    132 non-null    object 
 5   Asia_Other                                  132 non-null    int64  
 6   Asia_Total                                  132 non-null    object 
 7   Australia,NZ & Other Oceania_Passenger Car  132 non-null    object 
 8   Australia,NZ & Other Oceania_Pick up        132 non-null    object 
 9   Australia,NZ & Other Oceania_PPV            132 non-null    object 
 10  Australia,NZ & Other Oceania_Other          132 non-null    int64  
 11  Australia,NZ & Other Oceania_Total          132 non-null    object 
 12  Middle East_Passenger Car                   132 non-null    object 
 13  Middle East_Pick up                         132 non-null    object 
 14  Middle East_PPV                             132 non-null    object 
 15  Middle East_Other                           132 non-null    int64  
 16  Middle East_Total                           132 non-null    object 
 17  Africa_Passenger Car                        132 non-null    object 
 18  Africa_Pick up                              132 non-null    object 
 19  Africa_PPV                                  132 non-null    object 
 20  Africa_Other                                132 non-null    int64  
 21  Africa_Total                                132 non-null    object 
 22  Europe_Passenger Car                        132 non-null    object 
 23  Europe_Pick up                              132 non-null    object 
 24  Europe_PPV                                  132 non-null    object 
 25  Europe_Other                                132 non-null    int64  
 26  Europe_Total                                132 non-null    object 
 27  Central & South America_Passenger Car       132 non-null    object 
 28  Central & South America_Pick up             132 non-null    object 
 29  Central & South America_PPV                 132 non-null    object 
 30  Central & South America_Other               132 non-null    int64  
 31  Central & South America_Total               132 non-null    object 
 32  North America_Passenger Car                 132 non-null    object 
 33  North America_Pick up                       132 non-null    object 
 34  North America_PPV                           132 non-null    object 
 35  North America_Other                         132 non-null    int64  
 36  North America_Total                         132 non-null    object 
 37  Others_Passenger Car                        132 non-null    int64  
 38  Others_Pick up                              132 non-null    int64  
 39  Others_PPV                                  132 non-null    int64  
 40  Others_Other                                132 non-null    int64  
 41  Others_Total                                132 non-null    int64  
 42  Total_Passenger Car                         132 non-null    object 
 43  Total_Pick up                               132 non-null    object 
 44  Total_PPV                                   132 non-null    object 
 45  Total_Other                                 132 non-null    int64  
 46  Total_Total                                 132 non-null    object 
dtypes: float64(1), int64(13), object(33)
memory usage: 48.6+ KB
```
```
new_register = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/New_Car_with_Fuel.csv?raw=true")
new_register.info()
```
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
```
charger_df = pd.read_csv("https://github.com/crispyporkwithholybasil/5001-DADS-miniproject/blob/main/EV_Charger(x-).csv?raw=true",header = 2)
charger_df.info()
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 55 entries, 0 to 54
Data columns (total 52 columns):
 #   Column                     Non-Null Count  Dtype  
---  ------                     --------------  -----  
 0   ปี                         48 non-null     float64
 1   เดือน                      49 non-null     object 
 2   Unnamed: 2                 0 non-null      float64
 3   บ้านอยู่อาศัย              51 non-null     object 
 4   กิจการ    ขนาดเล็ก         49 non-null     object 
 5   กิจการ     ขนาดกลาง        49 non-null     object 
 6   กิจการ      ขนาดใหญ่       49 non-null     object 
 7   กิจการ     เฉพาะอย่าง      49 non-null     object 
 8   องค์กรที่ไม่แสวงหากำไร     49 non-null     object 
 9   สูบน้ำเพื่อการเกษตร        49 non-null     object 
 10  Unnamed: 10                50 non-null     object 
 11  สถานีอัดประจุไฟฟ้า         50 non-null     object 
 12      จำนวนผู้ใช้ไฟฟ้า       49 non-null     object 
 13  Unnamed: 13                0 non-null      float64
 14  Unnamed: 14                0 non-null      float64
 15  Unnamed: 15                0 non-null      float64
 16  Unnamed: 16                0 non-null      float64
 17  Unnamed: 17                0 non-null      float64
 18  Unnamed: 18                0 non-null      float64
 19  Unnamed: 19                0 non-null      float64
 20  Unnamed: 20                0 non-null      float64
 21  Unnamed: 21                0 non-null      float64
 22  Unnamed: 22                0 non-null      float64
 23  Unnamed: 23                0 non-null      float64
 24  Unnamed: 24                0 non-null      float64
 25  Unnamed: 25                0 non-null      float64
 26  Unnamed: 26                0 non-null      float64
 27  Unnamed: 27                0 non-null      float64
 28  Unnamed: 28                0 non-null      float64
 29  Unnamed: 29                0 non-null      float64
 30  Unnamed: 30                0 non-null      float64
 31  Unnamed: 31                0 non-null      float64
 32  Unnamed: 32                0 non-null      float64
 33  Unnamed: 33                0 non-null      float64
 34  Unnamed: 34                0 non-null      float64
 35  Unnamed: 35                0 non-null      float64
 36  Unnamed: 36                0 non-null      float64
 37  Unnamed: 37                0 non-null      float64
 38  Unnamed: 38                0 non-null      float64
 39  Unnamed: 39                0 non-null      float64
 40  Unnamed: 40                0 non-null      float64
 41  Unnamed: 41                0 non-null      float64
 42  Unnamed: 42                0 non-null      float64
 43  Unnamed: 43                0 non-null      float64
 44  Unnamed: 44                0 non-null      float64
 45  Unnamed: 45                0 non-null      float64
 46  Unnamed: 46                0 non-null      float64
 47  Unnamed: 47                1 non-null      float64
 48  Unnamed: 48                1 non-null      float64
 49  Unnamed: 49                1 non-null      float64
 50  Unnamed: 50                1 non-null      float64
 51  Unnamed: 51                1 non-null      float64
dtypes: float64(41), object(11)
memory usage: 22.5+ KB
```
```
#Import Thailand map data
#Thailand - Subnational Administrative Boundaries Dataframe from https://data.humdata.org/dataset/cod-ab-tha
gdf = gpd.read_file('tha_admbnda_adm1_rtsd_20190221.shp')
gdf.head()
```
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
df_charger 
```
2.2)เนื่องจากข้อมูลบางชุดไม่ได้อยู่ในรูปของ datetime format ดังนั้นจึงต้องมีการแปลงข้อมูลที่มีอยู่ในอยู่ในรูป dattime โดยการ map ข้อมูลจากเดือนภาษาไทยให้เป็นตัวเลขและสร้าง Column ['Year-Month']
และนำ Column ใหม่ที่สร้างขึ้นมาไปใช้งานต่อ

```python
month_map = {'มกราคม': 1, 'กุมภาพันธ์': 2, 'มีนาคม': 3, 'เมษายน': 4, 'พฤษภาคม': 5, 'มิถุนายน': 6, 'กรกฎาคม': 7, 'สิงหาคม': 8, 'กันยายน': 9, 'ตุลาคม': 10, 'พฤศจิกายน': 11, 'ธันวาคม': 12}
```

```
df_prod_dom_4.insert(2, 'Year-Month', pd.to_datetime(df_prod_dom_4[['Year', 'Month']].assign(Day=1)) )
df_prod_dom_4
```

2.2) ข้อมูลที่ได้รับมาอยู่ในรูปปี พุทธศักราช(B.E.) ส่งผลให้ไม่สามารถ แปลงข้อมูลให้อยู่ในรูปของ datetime format ได้ จึงต้องทำการแปลงให้เป็นคริสต์ศักราช (A.D.)
```
charger_df7 = charger_df6.rename(columns={'ปี': 'Year','เดือน':'Month'})
charger_df7['Year'] = charger_df7['Year']-543
charger_df7.insert(2, 'Year-Month', pd.to_datetime(charger_df7[['Year', 'Month']].assign(Day=1)) )
charger_df7
```

2.3) Remove whitespace and comma จาก dataframe
```
df_export_unit1['Month'] = df_export_unit1['Month'].str.replace(r"\s+", "")
df_export_unit2 = df_export_unit1
df_export_unit2
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


  ![image](https://user-images.githubusercontent.com/114766023/226210841-276c357a-2df6-402f-b173-95055f136a28.png)

3.4) สถานีอัดประจุไฟฟ้าซึ่งเป็นปัจจัยที่มีส่วนในการเติบโตของรถยนต์ไฟฟ้าในประเทศมีความสอดคลองกับการขยายตัวของความต้องใช้รถไฟฟ้าหรือไม่
 
![image](https://user-images.githubusercontent.com/114766023/226211114-d01f6d0c-f930-4789-ac66-14966ba3ee46.png)

![image](https://user-images.githubusercontent.com/114766023/226212639-69a0af9d-cf11-47bf-b9ea-62ebbed849be.png)

```

```
 ![image](https://user-images.githubusercontent.com/114766023/226211275-a2221f6a-e69f-41c6-8d43-8d55ac59f500.png)
 
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
