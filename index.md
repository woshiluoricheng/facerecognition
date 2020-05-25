# -*- coding: utf-8 -*-


# 导入人脸识别模块
from frsclient import FrsClient
from PIL.ImageTk import PhotoImage

# 构造客户端服务
# 输入华为云访问密匙
ak = "LNXEIAA43DBFTHWQQRGS"
sk = "1qDPgqhtke2tYEajSQXTkMx6FzAT0HUIm2LxJCtj"
# 输入项目id
project_id = "457663349e994c37b9e7400f8eb2c4b1"
# 通过之前的ak和sk以及projectid新建人脸识别对象
frs_client = FrsClient(ak=ak, sk=sk, project_id=project_id) 
#这些是要调用这个服务 要去申请这个服务 去华为官网申请做这个项目 申请账户和密码
#申请好了 所以用之前申请的ak sk project_id工程名 用这三个来保证你能获取到一个完整的项目


import cv2
cap = cv2.VideoCapture(0)    #cv2.VideoCapture(0)代表调取摄像头资源，其中0代表电脑摄像头，1代表外接摄像头(usb摄像头)

while(1):
    # 获得图片
    ret, frame = cap.read()
    # 展示图片
    cv2.imshow("capture", frame)
    """
       OxFF：是一个位掩码，一旦使用了掩码，就可以检查它是否是相应的值
       ord('q')：返回q对应的unicode码对应的值(113)
    """
    if cv2.waitKey(1) & 0xFF == ord('q'):
        #存储图片
        cv2.imwrite("camera.jpg", frame)
        break
#释放对象并销毁窗口
cap.release()
cv2.destroyAllWindows()


import tkinter as tk  #对话框的库
from tkinter import filedialog,Canvas
# 定义函数，用于选择到相应按钮时
def select_jpg1():
    global path1
    global Entry1
    # 获取path1的路径
    path1 = filedialog.askopenfilename()  #打开文件获取它的地址
    # 将path1的路径插入到Entry1中(先清空再插入)
    Entry1.delete(0,'end')
    Entry1.insert(tk.INSERT,path1)

def select_jpg2():
    global path2
    global Entry2
    # 获取path2的路径
    path2 = filedialog.askopenfilename()
    # 将path2的路径插入到Entry2中(先清空再插入)
    Entry2.delete(0, 'end')
    Entry2.insert(tk.INSERT,path2)
    
def getgetget():
    # 调用华为云人脸比对服务
    result_detect = frs_client.get_compare_service().compare_face_by_file(path1, path2)
    # 通过get_similarity()来获取置信度
    similarity = result_detect.get_similarity()
    print("两张人脸的相似度为：%f"% similarity)

    # 利用matplotlib绘图库和PIL图像处理库来绘制图像
    import matplotlib.pyplot as plt
    from PIL import Image,ImageDraw  #一个方框和一个图是用两个做的 不是一下子做出来的 不是同时出现的 
    #用PIL把人脸图贴在那个地方 然后再去画人脸的位置 人脸位置怎么去画呢 华为云有获得人脸位置的API 然后就有一个
    #result_detect.get_image1_bounding_box() 一个方形的边框 然后我们用绘图库绘图出来 pts就是四个点的坐标 就是一个矩形的坐标
    #开始绘图 绘制边框 显示图片 不显示坐标轴 画一个方形 没了 挺简单一个 因为用的是华为的API 大概是这个样子 具体华为API怎么用在华为官网是有写的


    # 创建图形，宽为8，高为6
    plt.figure(figsize=(8, 6))


    # 绘制第一个结果
    # 建立图像对象，边框对象，边框左上和右下的坐标
    im1 = Image.open(path1)
    bbox = result_detect.get_image1_bounding_box()
    pts = (bbox['top_left_x'], bbox['top_left_y'],bbox['top_left_x'] + bbox['width'], bbox['top_left_y'] + bbox['height'])

    # 设定颜色
    color = (0,0,255)
    # 绘制人脸图
    draw = ImageDraw.Draw(im1)
    # 绘制图形边框
    draw.rectangle(pts,outline=color,width=5)

    #读取并显示图片但不显示坐标轴
    plt.subplot(121)
    plt.imshow(im1)
    plt.axis('off')


    # 绘制第二个结果，方法同上
    im2 = Image.open(path2)
    bbox = result_detect.get_image2_bounding_box()
    pts = (bbox['top_left_x'], bbox['top_left_y'],bbox['top_left_x'] + bbox['width'], bbox['top_left_y'] + bbox['height'])
    color = (0,0,255)

    draw = ImageDraw.Draw(im2)
    draw.rectangle(pts,outline=color,width=5)

    plt.subplot(122)
    plt.imshow(im2)
    plt.axis('off')

    im1=im1.resize((150,150))
    im2=im2.resize((150,150))
    im1.save("save1.gif","GIF")
    im2.save("save2.gif","GIF")
    img1=PhotoImage(file='save1.gif')
    canvas.create_image(90,180,anchor='nw',image=img1)
    img2=PhotoImage(file='save2.gif')
    canvas.create_image(270,180,anchor='nw',image=img2)
    
    #设置标签
    result=tk.Label(text="两张人脸的相似度为：%f"% similarity)
    result.pack()
    result.place(x=50,y=350)
    
    #画上去
    root.mainloop()
    
# 初始化
root = tk.Tk()  #创建一个tk图形界面
root.resizable(height=False, width=False) #禁止修改窗体大小

# 设置画布(为后面添加图片准备)
canvas = Canvas(root,width=500,height=410)
canvas.pack()

# 设置初始参数
Entry1 = tk.Entry(root,bg='white',width=45) #空白文字画框entry1
Entry2 = tk.Entry(root,bg='white',width=45) #空白文字画框entry2
path1 = '' #图1的路径
path2 = '' #图2的路径

# 设置窗体标题
root.title('获取需要识别的图片')

# 设置窗口大小和位置
root.geometry('500x410+570+200')

# 设置三个按钮的文字，宽度，以及点击之后相应的命令
button1 = tk.Button(root, text='浏览第一张图片', width=15, command=select_jpg1) #command就是命令 select_jpg1是前面定义的函数
button2 = tk.Button(root, text='浏览第二张图片', width=15, command=select_jpg2)
button3 = tk.Button(root, text='两个图片都选择完毕', width=30,command=getgetget)


Entry1.pack()   #.pack相当于进行循环 保证点击几次都可以
Entry2.pack()
button1.pack()
button2.pack()
button3.pack()

# 设置按钮以及text的位置
button1.place(x=20,y=25) #用place来设置
button2.place(x=20,y=80)
button3.place(x=140,y=125)
Entry1.place(x=150,y=30)
Entry2.place(x=150,y=80)
root.mainloop()   #保证对话框能出来
