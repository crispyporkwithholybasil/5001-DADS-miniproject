# Mini-Project : สถานการณ์อุตสาหกรรมรถยนต์ไทย
สำหรับ Mini-project นี้จะประกอบไปด้วยการนำเสนอ 4 ส่วนได้กัน ได้แก่

1.  Source of Information (ชุดข้อมูลที่กลุ่มเราเลือกใช้)
2. Cleansing DATA 
3. Exploratory Data Analysis (EDA)
4. Summary and Suggestion

## 1 : Source of Information (ชุดข้อมูลที่กลุ่มเราเลือกใช้)
1.1) ข้อมูลจากศูนย์สารสนเทศยานยนต์
  - ข้อมูลการผลิตรถนยนต์ภายในประเทศไทย
  - ข้อมูลการส่งออกรถยนต์จากประเทศ
  - สถิติรถจดทะเบียนใหม่ จำแนกตามประเภทเชื้อเพลิง

1.2) ข้อมูลจาก



## 2 : Cleansing DATA 
2.1) map ข้อมูลจากเดือนภาษาไทยให้เป็นตัวเลขและสร้าง Column ['Year-Month']

```python
month_map = {'มกราคม': 1, 'กุมภาพันธ์': 2, 'มีนาคม': 3, 'เมษายน': 4, 'พฤษภาคม': 5, 'มิถุนายน': 6, 'กรกฎาคม': 7, 'สิงหาคม': 8, 'กันยายน': 9, 'ตุลาคม': 10, 'พฤศจิกายน': 11, 'ธันวาคม': 12}
```

```
df_prod_dom_4.insert(2, 'Year-Month', pd.to_datetime(df_prod_dom_4[['Year', 'Month']].assign(Day=1)) )
df_prod_dom_4
```


![image](https://user-images.githubusercontent.com/83213407/226184043-4bdaade1-91af-4633-9d35-932ee2f0f0d3.png)
