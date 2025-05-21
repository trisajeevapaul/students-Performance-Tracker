# students-Performance-Tracker
from pymongo import MongoClient
from datetime import datetime

# Connect to MongoDB
client = MongoClient("mongodb://localhost:27017/")
db = client["student_tracker"]
students_col = db["students"]

# Add student
def add_student(student_id, name):
    student = {
        "student_id": student_id,
        "name": name,
        "courses": []
    }
    students_col.insert_one(student)
    print(f"Added student: {name}")

# Enroll student in a course
def enroll_course(student_id, course_id, course_name):
    students_col.update_one(
        {"student_id": student_id},
        {"$push": {
            "courses": {
                "course_id": course_id,
                "course_name": course_name,
                "grades": [],
                "attendance": []
            }
        }}
    )
    print(f"Enrolled student {student_id} in course {course_name}")

# Record a grade
def add_grade(student_id, course_id, grade):
    students_col.update_one(
        {"student_id": student_id, "courses.course_id": course_id},
        {"$push": {"courses.$.grades": grade}}
    )
    print(f"Added grade {grade} for student {student_id} in course {course_id}")

# Record attendance
def mark_attendance(student_id, course_id, present=True):
    date_today = datetime.now().strftime("%Y-%m-%d")
    students_col.update_one(
        {"student_id": student_id, "courses.course_id": course_id},
        {"$push": {"courses.$.attendance": {
            "date": date_today,
            "present": present
        }}}
    )
    print(f"Marked attendance for student {student_id} in {course_id} on {date_today}: Present={present}")

# Calculate average grade per student per course
def calculate_average_grade(student_id, course_id):
    pipeline = [
        {"$match": {"student_id": student_id}},
        {"$unwind": "$courses"},
        {"$match": {"courses.course_id": course_id}},
        {"$project": {
            "course_name": "$courses.course_name",
            "avg_grade": {"$avg": "$courses.grades"}
        }}
    ]
    result = list(students_col.aggregate(pipeline))
    if result:
        print(f"Average grade in {result[0]['course_name']}: {result[0]['avg_grade']:.2f}")
    else:
        print("No data found.")

# Show student record
def show_student(student_id):
    student = students_col.find_one({"student_id": student_id})
    if student:
        print(f"\nStudent ID: {student['student_id']}\nName: {student['name']}")
        for course in student['courses']:
            print(f"\n  Course: {course['course_name']} ({course['course_id']})")
            print(f"    Grades: {course['grades']}")
            attendance = course['attendance']
            present_days = sum(1 for a in attendance if a['present'])
            print(f"    Attendance: {present_days}/{len(attendance)} days present")
    else:
        print("Student not found.")

# Sample run
if __name__ == "__main__":
    # Add and enroll students (first run only)
    # add_student("S001", "Alice Smith")
    # enroll_course("S001", "C001", "Math")
    # enroll_course("S001", "C002", "Science")

    # Add grades and attendance
    add_grade("S001", "C001", 85)
    add_grade("S001", "C001", 90)
    mark_attendance("S001", "C001", True)
    mark_attendance("S001", "C001", False)

    # Show records and average
    show_student("S001")
    calculate_average_grade("S001", "C001")


