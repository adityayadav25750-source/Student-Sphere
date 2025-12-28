# Student-Sphere
Student Management System project (Student sphere) is the project  which helps the user to store the hige data of their students to manage , retrive , stores information regarding students. 
# Student Management System (Flask + MySQL)
## Overview
A simple full-stack Student Management System built with Flask and MySQL. Features:
- Admin authentication
- Add/Edit/Delete students
- Attendance management
- Marks entry
- Dashboard
## Setup
1. Install Python 3.8+ and MySQL.
2. Create a database 'student_db' and run the provided SQL ('db_init.sql').
3. Update 'db_config' in 'app.py' with your DB credentials.
4. Install dependencies: 'pip install -r requirements.txt'
5. Run: 'python app.py"


Code 
from flask import Flask, render_template, request, redirect, url_for, session, flash
import mysql.connector
from functools import wraps

app = Flask(__name__)
app.secret_key = 'replace-with-a-secure-key'

# Database config - update with your credentials
db_config = {
    'host': 'localhost',
    'user': 'root',
    'password': '',
    'database': 'student_db'
}

def get_db():
    return mysql.connector.connect(**db_config)

def login_required(f):
    @wraps(f)
    def wrapped(*args, **kwargs):
        if 'logged_in' in session:
            return f(*args, **kwargs)
        return redirect(url_for('login'))
    return wrapped

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        db = get_db()
        cursor = db.cursor(dictionary=True)
        cursor.execute("SELECT * FROM admin WHERE username=%s AND password=%s", (username, password))
        user = cursor.fetchone()
        cursor.close()
        db.close()
        if user:
            session['logged_in'] = True
            session['username'] = username
            return redirect(url_for('dashboard'))
        else:
            flash('Invalid credentials', 'danger')
    return render_template('login.html')

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))

@app.route('/dashboard')
@login_required
def dashboard():
    db = get_db()
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM students")
    students = cursor.fetchall()
    cursor.execute("SELECT COUNT(*) as total FROM students")
    total = cursor.fetchone()['total']
    cursor.close()
    db.close()
    return render_template('dashboard.html', students=students, total=total)

@app.route('/students/add', methods=['GET', 'POST'])
@login_required
def add_student():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        clas = request.form['class']
        dob = request.form['dob']
        phone = request.form['phone']
        db = get_db()
        cursor = db.cursor()
        cursor.execute("INSERT INTO students (name, email, class, dob, phone) VALUES (%s,%s,%s,%s,%s)",
                       (name, email, clas, dob, phone))
        db.commit()
        cursor.close()
        db.close()
        flash('Student added successfully', 'success')
        return redirect(url_for('dashboard'))
    return render_template('add_student.html')

@app.route('/students/edit/<int:student_id>', methods=['GET', 'POST'])
@login_required
def edit_student(student_id):
    db = get_db()
    cursor = db.cursor(dictionary=True)
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        clas = request.form['class']
        dob = request.form['dob']
        phone = request.form['phone']
        cursor.execute("UPDATE students SET name=%s, email=%s, class=%s, dob=%s, phone=%s WHERE id=%s",
                       (name, email, clas, dob, phone, student_id))
        db.commit()
        cursor.close()
        db.close()
        flash('Student updated successfully', 'success')
        return redirect(url_for('dashboard'))
    cursor.execute("SELECT * FROM students WHERE id=%s", (student_id,))
    student = cursor.fetchone()
    cursor.close()
    db.close()
    return render_template('edit_student.html', student=student)

@app.route('/students/delete/<int:student_id>', methods=['POST'])
@login_required
def delete_student(student_id):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("DELETE FROM students WHERE id=%s", (student_id,))
    db.commit()
    cursor.close()
    db.close()
    flash('Student deleted', 'info')
    return redirect(url_for('dashboard'))

@app.route('/attendance', methods=['GET', 'POST'])
@login_required
def attendance():
    db = get_db()
    cursor = db.cursor(dictionary=True)
    if request.method == 'POST':
        student_id = request.form['student_id']
        date = request.form['date']
        status = request.form['status']
        cursor.execute("INSERT INTO attendance (student_id, date, status) VALUES (%s,%s,%s)",
                       (student_id, date, status))
        db.commit()
        flash('Attendance recorded', 'success')
        cursor.close()
        db.close()
        return redirect(url_for('attendance'))
    cursor.execute("SELECT a.*, s.name FROM attendance a JOIN students s ON a.student_id = s.id ORDER BY a.date DESC")
    records = cursor.fetchall()
    cursor.execute("SELECT id, name FROM students")
    students = cursor.fetchall()
    cursor.close()
    db.close()
    return render_template('attendance.html', records=records, students=students)

@app.route('/marks', methods=['GET', 'POST'])
@login_required
def marks():
    db = get_db()
    cursor = db.cursor(dictionary=True)
    if request...