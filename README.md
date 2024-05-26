# Attendance-Generator-model-
#This is a GUI based project which uses  tkinter module in Python for GUI purposes

# coding: utf-8

# In[ ]:


import tkinter as tk
from tkinter import *
from tkinter import messagebox as mess
from tkinter import ttk
import tkinter.simpledialog as tsd
import os
import cv2
import csv
import numpy as np
from PIL import Image
import pandas as pd
import datetime
import time


# In[ ]:


class App:
    def __init__(self):
        self.root=Tk()
        self.root.title("Face Recognition Based Attendance System")
        self.root.geometry("1280x720")
        self.root.resizable(True,True)
        self.root.configure(background='#355454')
        heading =Label(self.root, text="Face Recognition Based Attendance System" ,fg="white",bg="#355454" ,width=60 ,height=1,font=('times', 29, ' bold '))
        heading.place(x=10, y=10,relwidth=1)
        frame1 = Frame(self.root, bg="white")
        frame1.place(relx=0.11, rely=0.15, relwidth=0.39, relheight=0.80)

        frame2 = Frame(self.root, bg="white")
        frame2.place(relx=0.51, rely=0.15, relwidth=0.39, relheight=0.80)
        
        head1 = Label(frame1, text="Register New Student", fg="white",bg="black" ,font=('times', 17, ' bold ') )
        head1.place(x=0,y=0,relwidth=1)

        head2 = Label(frame2, text="Mark Student's attendance", fg="white",bg="black" ,font=('times', 17, ' bold ') )
        head2.place(x=0,y=0,relwidth=1)
        
        #registretion frame
        lbl = Label(frame1, text="Enter ID",width=20  ,height=1  ,fg="black"  ,bg="white" ,font=('times', 17, ' bold ') )
        lbl.place(x=0, y=55)

        self.txtId = Entry(frame1,width=32 ,fg="black",bg="#e1f2f2",highlightcolor="#00aeff",highlightthickness=3,font=('times', 15, ' bold '))
        self.txtId.place(x=55, y=88,relwidth=0.75)

        lbl2 = Label(frame1, text="Enter Name",width=20  ,fg="black"  ,bg="white" ,font=('times', 17, ' bold '))
        lbl2.place(x=0, y=140)

        self.txtName = Entry(frame1,width=32 ,fg="black",bg="#e1f2f2",highlightcolor="#00aeff",highlightthickness=3,font=('times', 15, ' bold ')  )
        self.txtName.place(x=55, y=173,relwidth=0.75)
        self.message = Label(frame1, text="" ,bg="white" ,fg="black"  ,width=39,height=1, activebackground = "yellow" ,font=('times', 16, ' bold '))
        self.message.place(x=7, y=500)
        self.message1 =Label(frame1, text="" ,bg="white" ,fg="black"  ,width=39 ,height=1, activebackground = "yellow" ,font=('times', 15, ' bold '))
        self.message1.place(x=7, y=300)
        lbl3 = Label(frame2, text="Attendance Table",width=20  ,fg="black"  ,bg="white"  ,height=1 ,font=('times', 17, ' bold '))
        lbl3.place(x=100, y=120)
        
        clearButton = Button(frame1, text="Clear", command=self.clear, fg="white", bg="#13059c", width=11, activebackground = "white", font=('times', 12, ' bold '))
        clearButton.place(x=55, y=230,relwidth=0.29)

        takeImg = Button(frame1, text="Take Images", command=self.TakeImages, fg="black", bg="#00aeff", width=34, height=1, activebackground = "white", font=('times', 16, ' bold '))
        takeImg.place(x=30, y=350,relwidth=0.89)

        trainImg = Button(frame1, text="Save Profile", command=self.psw, fg="black", bg="#00aeff", width=34, height=1, activebackground = "white", font=('times', 16, ' bold '))
        trainImg.place(x=30, y=430,relwidth=0.89)
        trackImg = Button(frame2, text="Take Attendance", command=self.TrackImages, fg="black", bg="#00aeff", height=1, activebackground = "white" ,font=('times', 16, ' bold '))
        trackImg.place(x=30,y=60,relwidth=0.89)
        
        style = ttk.Style()
        style.configure("mystyle.Treeview", highlightthickness=0, bd=0, font=('Calibri', 11)) # Modify the font of the body
        style.configure("mystyle.Treeview.Heading",font=('times', 13,'bold')) # Modify the font of the headings
        style.layout("mystyle.Treeview", [('mystyle.Treeview.treearea', {'sticky': 'nswe'})]) # Remove the borders
        self.tb= ttk.Treeview(frame2,height =13,columns = ('name','date','time'),style="mystyle.Treeview")
        self.tb.column('#0',width=82)
        self.tb.column('name',width=130)
        self.tb.column('date',width=133)
        self.tb.column('time',width=133)
        self.tb.grid(row=2,column=0,padx=(0,0),pady=(150,0),columnspan=4)
        self.tb.heading('#0',text ='ID')
        self.tb.heading('name',text ='NAME')
        self.tb.heading('date',text ='DATE')
        self.tb.heading('time',text ='TIME')
        #SCROLLBAR--------------------------------------------------
        scroll=ttk.Scrollbar(frame2,orient='vertical',command=self.tb.yview)
        scroll.grid(row=2,column=4,padx=(0,100),pady=(150,0),sticky='ns')
        self.tb.configure(yscrollcommand=scroll.set)
        
        self.countTotalRegistration()
        self.root.mainloop()
    def countTotalRegistration(self):
        #Display total registration----------
        res=0
        exists = os.path.isfile("StudentDetails\StudentDetails.csv")
        if exists:
            with open("StudentDetails\StudentDetails.csv", 'r') as f:
                reader = csv.reader(f)
                for i in reader:
                    res = res + 1
                res = (res // 2) - 1
        else:
            res = 0
        self.message.configure(text='Total Registrations : '+str(res))
    def clear(self):
        self.txtId.delete(0, 'end')
        self.txtName.delete(0, 'end')
        self.txtId.focus()
    def assure_path_exists(self,path):
        dir = os.path.dirname(path)
        if not os.path.exists(dir):
            os.makedirs(dir)
    def getImagesAndLabels(self,path):
        # get the path of all the files in the folder
        imagePaths = [os.path.join(path, f) for f in os.listdir(path)]
        # create empty face list
        faces = []
        # create empty ID list
        Ids = []
        # now looping through all the image paths and loading the Ids and the images
        for imagePath in imagePaths:
            # loading the image and converting it to gray scale
            pilImage = Image.open(imagePath).convert('L')
            # Now we are converting the PIL image into numpy array
            imageNp = np.array(pilImage, 'uint8')
            # getting the Id from the image
            ID = int(os.path.split(imagePath)[-1].split(".")[1])
            # extract the face from the training image sample
            faces.append(imageNp)
            Ids.append(ID)
        return faces, Ids
    def TrainImages(self):
        exists = os.path.isfile("haarcascade_frontalface_default.xml")
        if exists:
            pass
        else:
            mess._show(title='Cascade File missing', message='haarcascade_frontalface_default.xml include in path')
        self.assure_path_exists("Pass_Train/")
        recognizer = cv2.face_LBPHFaceRecognizer.create()
        harcascadePath = "haarcascade_frontalface_default.xml"
        detector = cv2.CascadeClassifier(harcascadePath)
        faces, ID = self.getImagesAndLabels("D:\\TrainingImages")
        try:
            recognizer.train(faces, np.array(ID))
        except:
            mess._show(title='No Registrations', message='Please Register someone first!!!')
            return
        recognizer.save("Pass_Train\Trainner.yml")
        res = "Profile Saved Successfully"
        self.message1.configure(text=res)
        self.message.configure(text='Total Registrations till now  : ' + str(ID[0]))
        
    def TakeImages(self):
        exists = os.path.isfile("haarcascade_frontalface_default.xml")
        if exists:
            pass
        else:
            mess._show(title='Cascade File missing', message='haarcascade_frontalface_default.xml include in path')
            
        columns = ['SERIAL NO.', '', 'ID', '', 'NAME']
        self.assure_path_exists("StudentDetails/")
        self.assure_path_exists("D:\\TrainingImages\\")
        serial = 0
        exists = os.path.isfile("StudentDetails\StudentDetails.csv")
        if exists:
            with open("StudentDetails\StudentDetails.csv", 'r') as csvFile1:
                reader1 = csv.reader(csvFile1)
                for l in reader1:
                    serial = serial + 1
                #serial = (serial // 2)
        else:
            with open("StudentDetails\StudentDetails.csv", 'a+', newline='') as csvFile1:
                writer = csv.writer(csvFile1)
                writer.writerow(columns)
                serial = 1
                
        Id = (self.txtId.get())
        name = (self.txtName.get())
        if ((name.isalpha()) or (' ' in name)):
            cam = cv2.VideoCapture(0)
            harcascadePath = "haarcascade_frontalface_default.xml"
            detector = cv2.CascadeClassifier(harcascadePath)
            sampleNum = 0
            while True:
                ret, img = cam.read()
                gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
                faces = detector.detectMultiScale(gray, 1.05, 5)
                for (x, y, w, h) in faces:
                    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)
                    sampleNum = sampleNum + 1
                    cv2.imwrite("D:\\TrainingImages\\" + name + "." + str(serial) + "." + Id + '.' + str(sampleNum) + ".jpg",gray[y:y + h, x:x + w])
                    cv2.imshow('Taking Images', img)
            # wait for 100 miliseconds
                if cv2.waitKey(100) & 0xFF == ord('q'):
                    break
            # break if the sample number is morethan 100
                elif sampleNum > 100:
                    break
            cam.release()
            cv2.destroyAllWindows()
            res = "Images Taken for ID : " + Id
            row = [serial, '', Id, '', name]
            with open('StudentDetails\StudentDetails.csv', 'a+') as csvFile:
                writer = csv.writer(csvFile)
                writer.writerow(row)
            self.message1.configure(text=res)
        else:
            if (name.isalpha() == False):
                res = "Enter Correct name"
                self.message1.configure(text=res)
    def TrackImages(self):
        exists = os.path.isfile("haarcascade_frontalface_default.xml")
        if exists:
            pass
        else:
            mess._show(title='Cascade File missing', message='haarcascade_frontalface_default.xml include in path')
        self.assure_path_exists("Attendance/")
        self.assure_path_exists("StudentDetails/")
        for k in self.tb.get_children():
            tb.delete(k)
        msg = ''
        i = 0
        j = 0
        recognizer =cv2.face.LBPHFaceRecognizer_create() 
        exists3 = os.path.isfile("Pass_Train\Trainner.yml")
        if exists3:
            recognizer.read("Pass_Train\Trainner.yml")
        else:
            mess._show(title='Data Missing', message='Please click on Save Profile to reset data!!')
            return
        harcascadePath = "haarcascade_frontalface_default.xml"
        faceCascade = cv2.CascadeClassifier(harcascadePath);

        cam = cv2.VideoCapture(0)
        font = cv2.FONT_HERSHEY_SIMPLEX
        col_names = ['Id', '', 'Name', '', 'Date', '', 'Time']
        exists1 = os.path.isfile("StudentDetails\StudentDetails.csv")
        if exists1:
            df = pd.read_csv("StudentDetails\StudentDetails.csv")
        else:
            mess._show(title='Details Missing', message='Students details are missing, please check!')
            cam.release()
            cv2.destroyAllWindows()
            window.destroy()
        while True:
            ret, im = cam.read()
            gray = cv2.cvtColor(im, cv2.COLOR_BGR2GRAY)
            faces = faceCascade.detectMultiScale(gray, 1.2, 5)
            for (x, y, w, h) in faces:
                cv2.rectangle(im, (x, y), (x + w, y + h), (255, 0, 0), 2)
                serial, conf = recognizer.predict(gray[y:y + h, x:x + w])
                if (conf < 50):
                    ts = time.time()
                    date = datetime.datetime.fromtimestamp(ts).strftime('%d-%m-%Y')
                    timeStamp = datetime.datetime.fromtimestamp(ts).strftime('%H:%M:%S')
                    aa = df.loc[df['SERIAL NO.'] == serial]['NAME'].values
                    ID = df.loc[df['SERIAL NO.'] == serial]['ID'].values
                    ID = str(ID)
                    ID = ID[1:-1]
                    bb = str(aa)
                    bb = bb[2:-2]
                    attendance = [str(ID), '', bb, '', str(date), '', str(timeStamp)]

                else:
                    Id = 'Unknown'
                    bb = str(Id)
                cv2.putText(im, str(bb), (x, y + h), font, 1, (0, 251, 255), 2)
            cv2.imshow('Taking Attendance', im)
            if (cv2.waitKey(1) == ord('q')):
                break
        ts = time.time()
        date = datetime.datetime.fromtimestamp(ts).strftime('%d-%m-%Y')
        exists = os.path.isfile("Attendance\Attendance_" + date + ".csv")
        if exists:
            with open("Attendance\Attendance_" + date + ".csv", 'a+') as csvFile1:
                writer = csv.writer(csvFile1)
                writer.writerow(attendance)
        else:
            with open("Attendance\Attendance_" + date + ".csv", 'a+') as csvFile1:
                writer = csv.writer(csvFile1)
                writer.writerow(col_names)
                writer.writerow(attendance)
        with open("Attendance\Attendance_" + date + ".csv", 'r') as csvFile1:
            reader1 = csv.reader(csvFile1)
            for lines in reader1:
                i = i + 1
                if (i > 1):
                    if (i % 2 != 0):
                        iidd = str(lines[0]) + '   '
                        self.tb.insert('', 0, text=iidd, values=(str(lines[2]), str(lines[4]), str(lines[6])))
        cam.release()
        cv2.destroyAllWindows()

    def psw(self):
        str_pass=None
        self.assure_path_exists("Pass_Train/")
        exists1 = os.path.isfile("Pass_Train\\pass.txt")
        if exists1:
            tf = open("Pass_Train\\pass.txt", "r")
            str_pass =new_pass= tf.read()
            tf.close()
        password = tsd.askstring('Password', 'Enter Password', show='*')
        if (password == str_pass):
            self.TrainImages()

        elif password == None:
            pass
        else:
            mess._show(title='Wrong Password', message='You have entered wrong password')
obj=App()


# In[ ]:





# In[ ]:






