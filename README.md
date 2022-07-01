# Palm_detection
![detect2000_2000](https://user-images.githubusercontent.com/106720151/176812700-82afcd08-7732-4011-8aed-fea6897ea9a4.png)

<img width="808" alt="result" src="https://user-images.githubusercontent.com/101788422/176816516-3a80458d-76ec-4c19-98c1-30dd48e44b7d.png">
>รูปจากโปรแกรม QGIS

# สร้างสภาพแวดล้อมใหม่
`code conda create <environment name> python=3.9`

`code conda activate <environment name>`
# Yolov5 เพื่อการเทรนโมเดล และ detect
 
สร้าง โฟลเดอร์ของ yolov5

`cd <folder>`

`git clone https://github.com/ultralytics/yolov5`

`pip install -r requirements.txt`


(file label อยู่ในรูป yolo format : (class no) (xmin) (ymin) (width) (height) )

#### ในโฟลเดอร์ ..\yolov5\data\coco.yaml จะต้องแก้
- Path  : path ใหญ่ที่สุด 
- Train : ใส่ folder train
- Val   : ใส่ folder val
- Test  : ใส่ folder test
- Nc    : จำนวน class 
- Names : ชื่อ class ทั้งหมดเป็น list

![image](https://user-images.githubusercontent.com/106720151/176808489-17a147cb-9ff1-49ad-80c1-4bf3881114ec.png)
> การวาง folder

![image](https://user-images.githubusercontent.com/106720151/176809755-dd416920-a1f1-49ac-80e3-0a4d44deb5ee.png)
> ตัวอย่างการใส่ file paths


จากนั้น run code ส่วนของ YOLOv5 สามารถ ซึ่ง YOLOv5 สามารถ detect ได้มากสุดแค่รูปขนาดประมาณไม่เกิน 2500*2500 pixels   หลังจาก train จะได้ไฟล์ Weight ชื่อ best.pt อยู่ในโฟลเดอร์ ../yolov5/run/train/exp/weight/…    ซึ่งจะนำไปใช้ต่อใน  SAHI library 

# SAHI เพื่อการ detect รูปขนาดใหญ่มาก ๆ
ติดตั้ง sahi ลงใน environment เดิม 

`pip install -U torch sahi yolov5`
-	แก้ตัวแปร yolov5_model_path ให้เป็น weight ที่ได้จากการเทรน 
-	ใช้คำสั่ง result.to_coco_annotations() จะได้เป็น list ที่เก็บ dict  ใน dict มี key ชื่อ bbox  เก็บค่า xmin,ymin,width,height  
-	run cell สุดท้ายเพื่อแปลงเป็นค่า center(x,y) แล้วนำไปใส่ในส่วน Rasterio
#### code ส่วนที่ต้องแก้
```python
yolov5_model_path = r'<path  to  weight>’
result = get_sliced_prediction(“<path to image>”,
                               detection_model,
                               slice_height =1024,
                               slice_width = 1024,
                               overlap_height_ratio = 0.2,
                               overlap_width_ratio = 0.2
                               )
```

# Rasterio แปลงค่าให้เป็นพิกัดในโลกจริง
ติดตั้ง library
`pip install rasterio `
หาก error ให้ติดตั้ง GDAL and rasterio wheel จาก https://www.lfd.uci.edu/~gohlke/pythonlibs/ แล้วค่อย pip3 install <filename>

```python
with rasterio.open("<path to image>") as dataset:
    print(dataset.profile)
    aff = dataset.transform

with open("<path with name of file to store csv>", "w") as f:
    writer = csv.writer(f)
    writer.writerow(["x","y"])
    for c in row_column: 
        writer.writerow(aff*c)
```
