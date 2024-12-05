from datetime import datetime
import pytz
from pymongo import MongoClient
from getpass import getpass

# Set Indian Standard Time (IST) timezone
IST = pytz.timezone('Asia/Kolkata')

# Connect to MongoDB
def connect_to_mongodb():
    client = MongoClient('mongodb+srv://nachammaiganesan01:Nachammai2001@tododb.zthr6.mongodb.net/?retryWrites=true&w=majority&appName=ToDoDb')
    
    db = client['todo_db']  # Specify your database
    users_collection = db['users_collection']  # Collection for users
    tasks_collection = db['tasks_collection']  # Collection for tasks
    return client, users_collection, tasks_collection

# User login or registration
def user_auth(users_collection):
    while True:
        print("\n1. Register\n2. Login")
        choice = input("Choose an option (1-2): ")
        
        if choice == '1':  # Registration
            username = input("Enter a username: ")
            password = getpass("Enter a password: ")  # Hide password input
            
            # Check if user already exists
            if users_collection.find_one({"username": username}):
                print("Username already exists, please try again.")
            else:
                users_collection.insert_one({"username": username, "password": password})
                print(f"User {username} registered successfully!")
                return username  # Return logged-in user

        elif choice == '2':  # Login
            username = input("Enter your username: ")
            password = getpass("Enter your password: ")

            user = users_collection.find_one({"username": username, "password": password})
            if user:
                print(f"Welcome back, {username}!")
                return username  # Return logged-in user
            else:
                print("Invalid username or password, please try again.")

        else:
            print("Invalid choice, please try again.")

# Show menu
def show_menu():
    menu = '''
To-Do List Menu
1. View To-Do List
2. Add Task
3. Remove Task
4. Exit '''
    print(menu)

# View tasks
def view_tasks(collection, username):
    tasks = list(collection.find({"username": username}))  # Fetch tasks for the current user
    if not tasks:
        print("\nYour To-Do list is empty.")
    else:
        print("\n")
        for i, task in enumerate(tasks, start=1):
            task_name = task.get('task_name', 'Unnamed Task')
            description = task.get('description', 'No description provided')
            completion_time = task.get('completion_time', 'No completion time set')
            print(f"{i}. {task_name} - {description} - Complete by: {completion_time}")

# Function to validate date and time input
def get_valid_datetime_input(prompt_message):
    while True:
        user_input = input(prompt_message)
        try:
            # Parse the input into a datetime object using the IST timezone
            completion_time = datetime.strptime(user_input, '%Y-%m-%d %H:%M').astimezone(IST)
            return completion_time
        except ValueError:
            print("Invalid date/time format. Please enter in 'YYYY-MM-DD HH:MM' format.")

# Add task
def add_task(collection, username):
    task_name = input("Enter a new task: ")
    description = input("Enter a new description: ")
    
    # Get the completion date and time from the user
    completion_time = get_valid_datetime_input("Enter the completion date and time (YYYY-MM-DD HH:MM): ")
    
    if task_name:  # Ensure task_name is not empty
        current_time = datetime.now(IST)  # Get current time in IST
        task = {
            "username": username,
            "task_name": task_name,
            "description": description,
            "created_at": current_time.strftime("%Y-%m-%d %H:%M:%S"),  # Store time in IST
            "updated_at": current_time.strftime("%Y-%m-%d %H:%M:%S"),  # Store time in IST
            "completion_time": completion_time.strftime("%Y-%m-%d %H:%M:%S")  # Store completion time in IST
        }
        collection.insert_one(task)
        print(f"Task '{task_name}' has been added to your list, to be completed by {completion_time}.")
    else:
        print("Task name cannot be empty.")

# Remove task
def remove_task(collection, username):
    view_tasks(collection, username)
    tasks = list(collection.find({"username": username}))
    
    try:
        task_number = int(input("Enter the task number to remove: "))
        if 1 <= task_number <= len(tasks):
            removed_task = tasks[task_number - 1]
            collection.delete_one({"_id": removed_task["_id"]})  # Remove task from MongoDB
            print(f'"{removed_task["task_name"]}" has been removed.')
        else:
            print("Invalid task number.")
    except ValueError:
        print("Please Enter a Valid Number.")

# Main function
def main():
    # Connect to MongoDB
    client, users_collection, tasks_collection = connect_to_mongodb()
    
    try:
        # User login or registration
        username = user_auth(users_collection)
        
        while True:
            show_menu()
            choice = input("Choose an option (1-4): ")
            if choice == "1":
                view_tasks(tasks_collection, username)
            elif choice == "2":
                add_task(tasks_collection, username)
            elif choice == "3":
                remove_task(tasks_collection, username)
            elif choice == "4":
                print("Good Bye!")
                break
            else:
                print("Invalid Choice, Please Try Again.")
    
    finally:
        # Ensure MongoDB connection is closed
        client.close()
        print("MongoDB connection closed.")

if __name__ == "__main__":
    main()
