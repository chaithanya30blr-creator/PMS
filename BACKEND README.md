# PMS BACKEND
import streamlit as st # You need to add this import
import psycopg2
import pandas as pd

def get_db_connection():
    """Establishes a connection to the PostgreSQL database and displays the error if it fails."""
    try:
        conn = psycopg2.connect(
            host="localhost",
            database="PMS Dashboard",
            user="postgres",  # <--- Make sure this is your correct user
            password="1234"  # <--- Make sure this is your correct password
        )
        return conn
    except Exception as e:
        st.error(f"❌ Database Connection Failed: {e}")
        return None

def get_all_users(conn):
    """Retrieves all users from the users table."""
    query = "SELECT user_id, name, role FROM users"
    df = pd.read_sql(query, conn)
    return df

def get_goals_for_user(conn, user_id):
    """Fetches goals assigned to or managed by a user."""
    query = """
    SELECT g.goal_id, g.goal_description, g.due_date, g.status, e.name AS employee_name
    FROM goals g
    JOIN users e ON g.employee_id = e.user_id
    WHERE g.employee_id = %s OR g.manager_id = %s
    ORDER BY g.due_date;
    """
    df = pd.read_sql(query, conn, params=(user_id, user_id))
    return df

def get_tasks_for_goal(conn, goal_id):
    """Fetches tasks for a specific goal."""
    query = "SELECT task_id, task_description, is_approved FROM tasks WHERE goal_id = %s"
    df = pd.read_sql(query, conn, params=(goal_id,))
    return df

def create_goal(conn, description, due_date, manager_id, employee_id):
    """Inserts a new goal into the goals table."""
    with conn.cursor() as cur:
        cur.execute(
            "INSERT INTO goals (goal_description, due_date, status, manager_id, employee_id) VALUES (%s, %s, 'Draft', %s, %s)",
            (description, due_date, manager_id, employee_id)
        )
        conn.commit()
    return "Goal created successfully!"

def create_task(conn, description, goal_id, employee_id):
    """Logs a new task for a goal."""
    with conn.cursor() as cur:
        cur.execute(
            "INSERT INTO tasks (task_description, goal_id, employee_id) VALUES (%s, %s, %s)",
            (description, goal_id, employee_id)
        )
        conn.commit()
    return "Task created successfully, awaiting manager approval!"

def update_goal_status(conn, goal_id, status):
    """Updates the status of a goal."""
    with conn.cursor() as cur:
        cur.execute("UPDATE goals SET status = %s WHERE goal_id = %s", (status, goal_id))
        conn.commit()
    return "Goal status updated!"

def approve_task(conn, task_id):
    """Marks a task as approved."""
    with conn.cursor() as cur:
        cur.execute("UPDATE tasks SET is_approved = TRUE WHERE task_id = %s", (task_id,))
        conn.commit()
    return "Task approved!"

def create_feedback(conn, goal_id, manager_id, employee_id, feedback_text):
    """Adds a feedback entry for a goal."""
    with conn.cursor() as cur:
        cur.execute(
            "INSERT INTO feedback (feedback_text, goal_id, manager_id, employee_id) VALUES (%s, %s, %s, %s)",
            (feedback_text, goal_id, manager_id, employee_id)
        )
        conn.commit()
    return "Feedback submitted!"

def get_feedback_for_goal(conn, goal_id):
    """Retrieves all feedback for a specific goal."""
    query = "SELECT feedback_text, created_at FROM feedback WHERE goal_id = %s ORDER BY created_at DESC"
    df = pd.read_sql(query, conn, params=(goal_id,))
    return df

def get_employee_performance_history(conn, employee_id):
    """Generates a performance history report for an employee."""
    query = """
    SELECT
        g.goal_description, g.due_date, g.status,
        f.feedback_text, f.created_at AS feedback_date
    FROM goals g
    LEFT JOIN feedback f ON g.goal_id = f.goal_id
    WHERE g.employee_id = %s
    ORDER BY g.due_date DESC, f.created_at DESC;
    """
    df = pd.read_sql(query, conn, params=(employee_id,))
    return df
[2:27 am, 20/09/2025] Ks Chaithanya: Asap so that I will study from it only
[7:09 am, 20/09/2025] Archee Xime 4: Han bhjti
[7:38 am, 20/09/2025] Archee Xime 4: 
