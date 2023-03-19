# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย
สำหรับ Mini-project นี้จะประกอบไปด้วยการนำเสนอ 4 ส่วนได้กัน ได้แก่

1.  Source of Information (ชุดข้อมูลที่กลุ่มเราเลือกใช้)
2.  Cleansing DATA 
3.  Question and Answer (EDA)
4.  Summary and Suggestion

## 1 : Source of Information (ชุดข้อมูลที่กลุ่มเราเลือกใช้)
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
  - ข้อมูลการผลิตรถนยนต์ภายในประเทศไทย
  - ข้อมูลผลิตรถนยนต์เพื่อการส่งออกรถยนต์จากประเทศ
  - สถิติรถจดทะเบียนใหม่จำแนกตามประเภทเชื้อเพลิง

1.2) ข้อมูลจาก



## 2 : DATA Cleansing 
2.1) เนื่องจากข้อมูลบางชุดไม่ได้อยู่ในรูปของ datetime format ดังนั้นจึงต้องมีการแปลงข้อมูลที่มีอยู่ในอยู่ในรูป dattime โดยการ map ข้อมูลจากเดือนภาษาไทยให้เป็นตัวเลขและสร้าง Column ['Year-Month']
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
