# bap
import tkinter as tk
from tkinter import messagebox, ttk
from PIL import Image, ImageTk
import sqlite3 as sqltor

entry_Student_name = None
entry_Student_grade = None
entry_School_name = None
entry_Parent_name = None
entry_Parent_phone_number = None
entry_Parent_emailid = None

def create_student_table():
    conn = sqltor.connect('students.db')
    cursor = conn.cursor()

    cursor.execute('''CREATE TABLE IF NOT EXISTS student (
                    s_id INTEGER PRIMARY KEY,
                    s_name TEXT,
                    s_grade INTEGER,
                    s_school_name TEXT,
                    s_parent_name TEXT,
                    s_parent_phone_number INTEGER,
                    s_parent_email_id TEXT
                )''')

    conn.commit()
    conn.close()

def add_student():
    print("Student added")
    student_name = entry_Student_name.get()
    student_grade = int(entry_Student_grade.get())
    school_name = entry_School_name.get()
    parent_name = entry_Parent_name.get()
    parent_phone_number = int(entry_Parent_phone_number.get())
    parent_emailid = entry_Parent_emailid.get()
    
    conn = sqltor.connect('students.db')
    cursor = conn.cursor()

    sql = "INSERT INTO student(s_name, s_grade, s_school_name, s_parent_name, s_parent_phone_number, s_parent_email_id) VALUES(?, ?, ?, ?, ?, ?)"
    val = (student_name, student_grade, school_name, parent_name,parent_phone_number, parent_emailid)
    
    try:
        cursor.execute(sql, val)
        conn.commit()
        output_msg = f"{student_name}, {student_grade}, {school_name}, {parent_name}, {parent_phone_number}, {parent_emailid}, added successfully"
        label_status.configure(text=output_msg)
        print(output_msg)
    except Exception as e:
        conn.rollback()
        output_msg = f"Error: {e}"
        label_status.configure(text=output_msg)
        print(output_msg)
    finally:
        conn.close()
    
    entry_Student_name.delete(0, tk.END)
    entry_Student_grade.delete(0, tk.END)
    entry_School_name.delete(0, tk.END)
    entry_Parent_name.delete(0, tk.END)
    entry_Parent_phone_number.delete(0, tk.END)
    entry_Parent_emailid.delete(0, tk.END)

    display_all_students()

def display_all_students():
    for item in treeview_display_students.get_children():
        treeview_display_students.delete(item)
    
    conn = sqltor.connect('students.db')
    cursor = conn.cursor()

    cursor.execute('SELECT * FROM student')
    list_of_students = cursor.fetchall() 

    for student_record in list_of_students:
        treeview_display_students.insert(parent='', index='end', values=student_record[1:])

def delete_student_by_phone_number():
    phone_number = entry_Parent_phone_number.get()
    
    conn = sqltor.connect('students.db')
    cursor = conn.cursor()

    sqlquery = 'DELETE FROM student WHERE s_parent_phone_number = ?;'
    value = (phone_number,)
    cursor.execute(sqlquery, value)

    conn.commit()
    conn.close()

    display_all_students()

def search_student_name():
    for item in treeview_display_students.get_children():
        treeview_display_students.delete(item)

    name_input = entry_Student_name.get()
    
    conn = sqltor.connect('students.db')
    cursor = conn.cursor()

    sqlquery = 'SELECT * FROM student WHERE s_name LIKE ?;'
    value = [name_input + '%']
    cursor.execute(sqlquery, value)

    list_of_students = cursor.fetchall()
    
    for student_record in list_of_students:
        treeview_display_students.insert(parent='', index='end', values=student_record[1:])

    entry_Student_name.delete(0, tk.END)



def open_enquiry_management():
    enquiry_management_window = tk.Toplevel(root)
    enquiry_management_window.geometry('600x400')
    enquiry_management_window.configure(background='cyan')

    frame_status = tk.Frame(master=enquiry_management_window)
    frame_status.pack(pady=10)

    label_frame_display_students = tk.LabelFrame(master=enquiry_management_window, text='Display students')
    label_frame_display_students.pack(fill='both', expand='yes', padx=10, pady=10)

    label_frame_input = tk.LabelFrame(master=enquiry_management_window, text='Enter the student information')
    label_frame_input.pack(fill='both', expand='yes', padx=10, pady=10)

    frame_input = tk.Frame(master=enquiry_management_window, background='cyan', padx=10, pady=10)
    frame_input.pack(pady=10)

    label_frame_operations = tk.LabelFrame(master=enquiry_management_window, text='Operations')
    label_frame_operations.pack(fill='both', expand='yes', padx=10, pady=10)

    global treeview_display_students
    treeview_display_students = ttk.Treeview(master=label_frame_display_students, columns=(1, 2, 3, 4, 5, 6), show='headings')
    treeview_display_students.heading(1, text='SNAME')
    treeview_display_students.heading(2, text='GRADE')
    treeview_display_students.heading(3, text='S_SCHOOL_NAME')
    treeview_display_students.heading(4, text='S_PARENT_NAME')
    treeview_display_students.heading(5, text='S_PARENT_PHONE_NUMBER')
    treeview_display_students.heading(6, text='S_PARENT_EMAIL_ID')
    treeview_display_students.pack(fill='both', expand='yes', padx=10, pady=10)

    button_display_all_students = tk.Button(master=label_frame_operations, text='Display all students', command=display_all_students)
    button_display_all_students.pack()

    frame_buttons = tk.Frame(master=label_frame_operations)
    frame_buttons.pack()

    button_add_student = tk.Button(master=frame_buttons, text='Add student', command=add_student)
    button_search_student = tk.Button(master=frame_buttons, text='Search student', command=search_student_name)
    button_delete_student = tk.Button(master=frame_buttons, text='Delete student', command=delete_student_by_phone_number)

    button_add_student.grid(row=0, column=0, padx=5)
    button_search_student.grid(row=0, column=1, padx=5)
    button_delete_student.grid(row=0, column=2, padx=5)

    label_frame_Student_details = tk.LabelFrame(master=label_frame_input, text='STUDENT DETAILS')
    label_frame_Student_details.pack(side=tk.LEFT, expand='yes', fill='both')

    label_frame_Parent_details = tk.LabelFrame(master=label_frame_input, text='PARENT DETAILS')
    global entry_Student_name, entry_Student_grade, entry_School_name, entry_Parent_name, entry_Parent_phone_number, entry_Parent_emailid
    label_Student_name = tk.Label(master=label_frame_Student_details, text='ENTER THE NAME')
    entry_Student_name = tk.Entry(master=label_frame_Student_details)
    label_Student_grade = tk.Label(master=label_frame_Student_details, text='ENTER YOUR GRADE')
    entry_Student_grade = tk.Entry(master=label_frame_Student_details)
    label_School_name = tk.Label(master=label_frame_Student_details, text='ENTER YOUR SCHOOL NAME')
    entry_School_name = tk.Entry(master=label_frame_Student_details)
    label_Parent_name = tk.Label(master=label_frame_Parent_details, text='PARENT NAME')
    entry_Parent_name = tk.Entry(master=label_frame_Parent_details)
    label_Parent_phone_number = tk.Label(master=label_frame_Parent_details, text='PARENT PHONE NUMBER')
    entry_Parent_phone_number = tk.Entry(master=label_frame_Parent_details)
    label_Parent_emailid = tk.Label(master=label_frame_Parent_details, text='PARENT EMAIL ID')
    entry_Parent_emailid = tk.Entry(master=label_frame_Parent_details)
    entry_Parent_emailid = tk.Entry(master=label_frame_Parent_details)
    label_status = tk.Label(master=frame_status, text="Enter the details and click on add button")

    label_frame_Student_details.pack(side=tk.LEFT, expand='yes', fill='both')
    label_frame_Parent_details.pack(side=tk.LEFT, expand='yes', fill='both')
    label_Student_name.grid(row=1, column=0, sticky=tk.E)
    entry_Student_name.grid(row=1, column=1)
    label_Student_grade.grid(row=2, column=0, sticky=tk.E)
    entry_Student_grade.grid(row=2, column=1)
    label_School_name.grid(row=3, column=0, sticky=tk.E)
    entry_School_name.grid(row=3, column=1)
    label_Parent_name.grid(row=1, column=0, sticky=tk.E)
    entry_Parent_name.grid(row=1, column=1)
    label_Parent_phone_number.grid(row=2, column=0, sticky=tk.E)
    entry_Parent_phone_number.grid(row=2, column=1)
    label_Parent_emailid.grid(row=3, column=0, sticky=tk.E)
    entry_Parent_emailid.grid(row=3, column=1)

    label_status.pack()

import sqlite3
def create_database():
    mydb = sqlite3.connect('g71s20.db')
    cursor = mydb.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS course (
                        s_name TEXT,
                        s_age INTEGER,
                        course_name TEXT,
                        duration INTEGER,
                        s_phone_number INTEGER,
                        s_email_id TEXT
                    )''')
    mydb.commit()
    mydb.close()

def add_trainee():
    s_name = entry_trainee_name.get()
    s_age = int(entry_trainee_age.get())
    course_name = entry_trainee_course_name.get()
    duration = int(entry_duration.get())
    s_phone_number = int(entry_Parent_phone_number.get())
    s_email_id = entry_Parent_emailid.get()
    
    mydb = sqlite3.connect('g71s20.db')
    cursor = mydb.cursor()
    cursor.execute("INSERT INTO course (s_name, s_age, course_name, duration, s_phone_number, s_email_id) VALUES (?, ?, ?, ?, ?, ?)",
                   (s_name, s_age, course_name, duration, s_phone_number, s_email_id))
    mydb.commit()
    mydb.close()

    output_msg = f"{s_name}, {s_age}, {course_name}, {duration}, {s_phone_number}, {s_email_id}\nadded successfully"
    label_status.configure(text=output_msg)

def display_all_trainees():
    mydb = sqlite3.connect('g71s20.db')
    cursor = mydb.cursor()
    cursor.execute("SELECT * FROM course")
    list_of_students = cursor.fetchall()
    mydb.close()

    treeview_display_trainees.delete(*treeview_display_trainees.get_children())
    for trainee_record in list_of_students:
        treeview_display_trainees.insert('', 'end', values=trainee_record)

def search_name():
    name_input = entry_trainee_name.get()
    mydb = sqlite3.connect('g71s20.db')
    cursor = mydb.cursor()
    cursor.execute("SELECT * FROM course WHERE s_name LIKE ?", (name_input + '%',))
    list_of_students = cursor.fetchall()
    mydb.close()

    treeview_display_trainees.delete(*treeview_display_trainees.get_children())
    for trainee_record in list_of_students:
        treeview_display_trainees.insert('', 'end', values=trainee_record)

def delete_trainee_by_phone_number():
    phone_number = entry_Parent_phone_number.get()
    
    mydb = sqlite3.connect('g71s20.db')
    cursor = mydb.cursor()

    sqlquery = 'DELETE FROM course WHERE s_phone_number = ?;'
    value = (phone_number,)
    cursor.execute(sqlquery, value)

    mydb.commit()
    mydb.close()

    display_all_trainees()



def open_trainee_management():
    trainee_management_window = tk.Toplevel(root)
    trainee_management_window.geometry('600x400')
    trainee_management_window.configure(background='cyan')

    frame_status = tk.Frame(master=trainee_management_window)
    frame_status.pack(pady=10)

    label_frame_display_trainees = tk.LabelFrame(master=trainee_management_window, text='Display trainees')
    label_frame_display_trainees.pack(fill='both', expand='yes', padx=10, pady=10)

    label_frame_input = tk.LabelFrame(master=trainee_management_window, text='Enter the trainee information')
    label_frame_input.pack(fill='both', expand='yes', padx=10, pady=10)

    frame_input = tk.Frame(master=trainee_management_window, background='cyan', padx=10, pady=10)
    frame_input.pack(pady=10)

    label_frame_operations = tk.LabelFrame(master=trainee_management_window, text='Operations')
    label_frame_operations.pack(fill='both', expand='yes', padx=10, pady=10)

    global treeview_display_trainees
    treeview_display_trainees = ttk.Treeview(master=label_frame_display_trainees, columns=(1, 2, 3, 4, 5, 6), show='headings')
    treeview_display_trainees.heading(1, text='SNAME')
    treeview_display_trainees.heading(2, text='AGE')
    treeview_display_trainees.heading(3, text='COURSE_NAME')
    treeview_display_trainees.heading(4, text='DURATION')
    treeview_display_trainees.heading(5, text='S_PARENT_PHONE_NUMBER')
    treeview_display_trainees.heading(6, text='S_PARENT_EMAIL_ID')
    treeview_display_trainees.pack(fill='both', expand='yes', padx=10, pady=10)

    button_display_all_trainees = tk.Button(master=label_frame_operations, text='Display all trainees', command=display_all_trainees)
    button_display_all_trainees.pack()

    frame_buttons = tk.Frame(master=label_frame_operations)
    frame_buttons.pack()

    button_add_trainee = tk.Button(master=frame_buttons, text='Add trainee', command=add_trainee)  # Corrected function name
    button_search_trainee = tk.Button(master=frame_buttons, text='Search trainee', command=search_name)  # Corrected function name
    button_delete_trainee = tk.Button(master=frame_buttons, text='Delete trainee', command=delete_trainee_by_phone_number)  # Corrected function name

    button_add_trainee.grid(row=0, column=0, padx=5)
    button_search_trainee.grid(row=0, column=1, padx=5)
    button_delete_trainee.grid(row=0, column=2, padx=5)

    label_frame_trainee_details = tk.LabelFrame(master=label_frame_input, text='STUDENT DETAILS')
    label_frame_trainee_details.pack(side=tk.LEFT, expand='yes', fill='both')

    label_frame_Parent_details = tk.LabelFrame(master=label_frame_input, text='PARENT DETAILS')
    global entry_trainee_name, entry_trainee_age, entry_trainee_course_name, entry_duration, entry_Parent_phone_number, entry_Parent_emailid
    label_trainee_name = tk.Label(master=label_frame_trainee_details, text='ENTER THE NAME')
    entry_trainee_name = tk.Entry(master=label_frame_trainee_details)
    label_trainee_age = tk.Label(master=label_frame_trainee_details, text='ENTER YOUR AGE')  # Changed label text
    entry_trainee_age = tk.Entry(master=label_frame_trainee_details)
    label_trainee_course_name = tk.Label(master=label_frame_trainee_details, text='ENTER COURSE NAME')  # Changed label text
    entry_trainee_course_name = tk.Entry(master=label_frame_trainee_details)
    label_duration = tk.Label(master=label_frame_Parent_details, text='ENTER DURATION')  # Changed label text
    entry_duration = tk.Entry(master=label_frame_Parent_details)
    label_Parent_phone_number = tk.Label(master=label_frame_Parent_details, text='ENTER PHONE NUMBER')  # Changed label text
    entry_Parent_phone_number = tk.Entry(master=label_frame_Parent_details)
    label_Parent_emailid = tk.Label(master=label_frame_Parent_details, text='ENTER EMAIL ID')  # Changed label text
    entry_Parent_emailid = tk.Entry(master=label_frame_Parent_details)
    entry_Parent_emailid = tk.Entry(master=label_frame_Parent_details)
    label_status = tk.Label(master=frame_status, text="Enter the details and click on add button")

    label_frame_trainee_details.pack(side=tk.LEFT, expand='yes', fill='both')
    label_frame_Parent_details.pack(side=tk.LEFT, expand='yes', fill='both')
    label_trainee_name.grid(row=1, column=0, sticky=tk.E)
    entry_trainee_name.grid(row=1, column=1)
    label_trainee_age.grid(row=2, column=0, sticky=tk.E)
    entry_trainee_age.grid(row=2, column=1)
    label_trainee_course_name.grid(row=3, column=0, sticky=tk.E)
    entry_trainee_course_name.grid(row=3, column=1)
    label_duration.grid(row=1, column=0, sticky=tk.E)
    entry_duration.grid(row=1, column=1)
    label_Parent_phone_number.grid(row=2, column=0, sticky=tk.E)
    entry_Parent_phone_number.grid(row=2, column=1)
    label_Parent_emailid.grid(row=3, column=0, sticky=tk.E)
    entry_Parent_emailid.grid(row=3, column=1)

    label_status.pack()


def on_home_click():
    messagebox.showinfo("Home", "Home button clicked")

def on_profile_click():
    messagebox.showinfo("Profile", "Profile button clicked")

def on_settings_click():
    messagebox.showinfo("Settings", "Settings button clicked")

def on_more_info_click():
    messagebox.showinfo("More Info", "You clicked the 'More Info' button")
    

def resize_image(event):
    new_width = event.width
    new_height = event.height
    image = background_image.resize((new_width, new_height), Image.LANCZOS)
    background_photo = ImageTk.PhotoImage(image)
    content_canvas.create_image(0, 0, image=background_photo, anchor='nw')
    content_canvas.image = background_photo  # Keep a reference to avoid garbage collection

root = tk.Tk()
root.title("Navigation Bar Example with Background Image")
root.geometry("600x400")

nav_frame = tk.Frame(root, bg="#4a4a4a", height=50)
nav_frame.pack(fill='x')

home_button = tk.Button(nav_frame, text="Home", command=on_home_click, bg="#4a4a4a", fg="white", bd=0, font=("Arial", 14))
home_button.pack(side='left', padx=10, pady=10)

profile_button = tk.Button(nav_frame, text="Profile", command=on_profile_click, bg="#4a4a4a", fg="white", bd=0, font=("Arial", 14))
profile_button.pack(side='left', padx=10, pady=10)

settings_button = tk.Button(nav_frame, text="Settings", command=on_settings_click, bg="#4a4a4a", fg="white", bd=0, font=("Arial", 14))
settings_button.pack(side='left', padx=10, pady=10)

content_frame = tk.Frame(root, bg="cyan")
content_frame.place(relx=0, rely=0.1, relwidth=1, relheight=0.9)

sub_frame = tk.Frame(content_frame, bg="cyan")
sub_frame.place(relx=0.1, rely=0.1, relwidth=0.8, relheight=0.8)

heading_label = tk.Label(sub_frame, text="What is Skill Sync?", font=("Arial", 26), bg="peach puff")
heading_label.pack(pady=20)

description_frame = tk.Frame(sub_frame, bg="cyan")
description_frame.pack(pady=20, fill='both', expand=True)

description_label = tk.Label(description_frame, text="Skill Sync is a platform where students can enroll in various courses offered by our training center. Whether you're looking to improve your technical skills, learn a new language, or enhance your professional capabilities, Skill Sync has a course for you. Simply browse through our course offerings, find the one that suits your needs, and enroll online. Our experienced instructors and comprehensive curriculum ensure that you get the best learning experience.", font=("Arial", 14, "italic"), bg="peach puff", wraplength=1000, justify='left')
description_label.pack(padx=20, pady=20)



content_canvas = tk.Canvas(sub_frame)
content_canvas.pack(fill='both', expand=True)
content_canvas.bind('<Configure>', resize_image)

background_image = Image.open("training.jpg")

width = content_canvas.winfo_width()
height = content_canvas.winfo_height()
resized_image = background_image.resize((width, height), Image.LANCZOS)
background_photo = ImageTk.PhotoImage(resized_image)

content_canvas.create_image(0, 0, image=background_photo, anchor='nw')
content_canvas.image = background_photo

tile_button = tk.Button(content_canvas, text="Enquiry Management", command=open_enquiry_management, bg="white", fg="black", bd=0, font=("Arial", 12),
                        relief='raised', borderwidth=2, highlightbackground='black', padx=20, pady=10)
tile_button.pack(pady=50)

center_button = tk.Button(content_canvas, text="Trainee Management", command=open_trainee_management, bg="white", fg="black", bd=0, font=("Arial", 12),
                        relief='raised', borderwidth=2, highlightbackground='black', padx=20, pady=10)
center_button.pack(pady=30)

root.mainloop()
