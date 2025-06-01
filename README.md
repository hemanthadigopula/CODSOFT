# CODSOFT
#!/usr/bin/env python3
"""
Personal Task Manager
Created for CodSoft Internship Program
A simple yet effective command-line to-do list application
"""

import json
import os
from datetime import datetime
import sys

class TaskManager:
    def __init__(self):
        self.tasks_file = "my_tasks.json"
        self.tasks = self.load_tasks()
        self.task_counter = self.get_next_id()
    
    def load_tasks(self):
        """Load tasks from file or create empty list if file doesn't exist"""
        if os.path.exists(self.tasks_file):
            try:
                with open(self.tasks_file, 'r') as file:
                    return json.load(file)
            except (json.JSONDecodeError, FileNotFoundError):
                return []
        return []
    
    def save_tasks(self):
        """Save current tasks to file"""
        try:
            with open(self.tasks_file, 'w') as file:
                json.dump(self.tasks, file, indent=2)
        except Exception as e:
            print(f"Oops! Couldn't save tasks: {str(e)}")
    
    def get_next_id(self):
        """Get the next available task ID"""
        if not self.tasks:
            return 1
        return max(task['id'] for task in self.tasks) + 1
    
    def add_task(self, description, priority="medium"):
        """Add a new task to the list"""
        if not description.strip():
            print("Hey! You can't add an empty task. Please write something meaningful.")
            return
        
        new_task = {
            'id': self.task_counter,
            'description': description.strip(),
            'completed': False,
            'priority': priority.lower(),
            'created_date': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            'completed_date': None
        }
        
        self.tasks.append(new_task)
        self.task_counter += 1
        self.save_tasks()
        print(f"✅ Great! Added task #{new_task['id']}: '{description}'")
    
    def view_tasks(self, show_completed=True):
        """Display all tasks in a nice format"""
        if not self.tasks:
            print("🎉 Awesome! No tasks found. You're all caught up!")
            return
        
        print("\n" + "="*60)
        print("           📋 YOUR PERSONAL TASK LIST 📋")
        print("="*60)
        
        pending_tasks = [task for task in self.tasks if not task['completed']]
        completed_tasks = [task for task in self.tasks if task['completed']]
        
        if pending_tasks:
            print("\n🔥 PENDING TASKS:")
            print("-" * 40)
            for task in sorted(pending_tasks, key=lambda x: self.priority_order(x['priority'])):
                priority_emoji = self.get_priority_emoji(task['priority'])
                print(f"  {priority_emoji} #{task['id']} - {task['description']}")
                print(f"    📅 Created: {task['created_date']}")
                print()
        
        if completed_tasks and show_completed:
            print("\n✅ COMPLETED TASKS:")
            print("-" * 40)
            for task in completed_tasks:
                print(f"  ✔️  #{task['id']} - {task['description']}")
                if task['completed_date']:
                    print(f"    🎯 Completed: {task['completed_date']}")
                print()
        
        print(f"\n📊 Summary: {len(pending_tasks)} pending, {len(completed_tasks)} completed")
    
    def priority_order(self, priority):
        """Return order for priority sorting"""
        order = {'high': 1, 'medium': 2, 'low': 3}
        return order.get(priority, 2)
    
    def get_priority_emoji(self, priority):
        """Get emoji for priority level"""
        emojis = {'high': '🔴', 'medium': '🟡', 'low': '🟢'}
        return emojis.get(priority, '🟡')
    
    def mark_complete(self, task_id):
        """Mark a task as completed"""
        task = self.find_task(task_id)
        if not task:
            print(f"Hmm, I couldn't find task #{task_id}. Double-check the ID!")
            return
        
        if task['completed']:
            print(f"Task #{task_id} is already completed! 🎉")
            return
        
        task['completed'] = True
        task['completed_date'] = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.save_tasks()
        print(f"🎉 Woohoo! Marked task #{task_id} as completed: '{task['description']}'")
    
    def remove_task(self, task_id):
        """Remove a task completely"""
        task = self.find_task(task_id)
        if not task:
            print(f"Can't find task #{task_id}. Are you sure that's the right ID?")
            return
        
        description = task['description']
        self.tasks.remove(task)
        self.save_tasks()
        print(f"🗑️  Removed task #{task_id}: '{description}'")
    
    def find_task(self, task_id):
        """Find a task by its ID"""
        try:
            task_id = int(task_id)
            for task in self.tasks:
                if task['id'] == task_id:
                    return task
        except ValueError:
            pass
        return None
    
    def update_task(self, task_id, new_description):
        """Update task description"""
        task = self.find_task(task_id)
        if not task:
            print(f"Couldn't locate task #{task_id}. Check the ID again!")
            return
        
        old_desc = task['description']
        task['description'] = new_description.strip()
        self.save_tasks()
        print(f"📝 Updated task #{task_id}!")
        print(f"   Old: '{old_desc}'")
        print(f"   New: '{new_description}'")
    
    def search_tasks(self, keyword):
        """Search for tasks containing keyword"""
        matching_tasks = [task for task in self.tasks 
                         if keyword.lower() in task['description'].lower()]
        
        if not matching_tasks:
            print(f"No tasks found containing '{keyword}'. Try different keywords!")
            return
        
        print(f"\n🔍 Found {len(matching_tasks)} task(s) containing '{keyword}':")
        print("-" * 50)
        for task in matching_tasks:
            status = "✅" if task['completed'] else "⏳"
            priority_emoji = self.get_priority_emoji(task['priority'])
            print(f"  {status} {priority_emoji} #{task['id']} - {task['description']}")
        print()

def display_help():
    """Show available commands"""
    help_text = """
    🚀 PERSONAL TASK MANAGER - HELP GUIDE 🚀
    
    Available Commands:
    
    📝 add <task> [priority]     - Add a new task (priority: high/medium/low)
       Example: add "Complete Python assignment" high
    
    📋 list                      - Show all tasks
    📋 list active              - Show only pending tasks
    
    ✅ done <id>                 - Mark task as completed
       Example: done 5
    
    🗑️  remove <id>              - Delete a task permanently
       Example: remove 3
    
    ✏️  update <id> <new_task>   - Update task description
       Example: update 2 "Study for midterm exam"
    
    🔍 search <keyword>          - Search tasks by keyword
       Example: search assignment
    
    ❓ help                      - Show this help guide
    👋 quit                      - Exit the application
    
    💡 Tips:
    - Task IDs are shown as #1, #2, etc.
    - High priority tasks appear with 🔴, medium with 🟡, low with 🟢
    - Completed tasks show ✅, pending tasks show ⏳
    """
    print(help_text)

def main():
    """Main application loop"""
    task_manager = TaskManager()
    
    print("🎯 Welcome to Your Personal Task Manager!")
    print("Type 'help' to see available commands or 'quit' to exit.")
    print("Let's get organized and boost your productivity! 💪\n")
    
    while True:
        try:
            user_input = input("📝 What would you like to do? ").strip()
            
            if not user_input:
                continue
            
            # Parse command and arguments
            parts = user_input.split(' ', 1)
            command = parts[0].lower()
            args = parts[1] if len(parts) > 1 else ""
            
            if command in ['quit', 'exit', 'q']:
                print("👋 Thanks for using Task Manager! Stay productive!")
                break
            
            elif command in ['help', 'h', '?']:
                display_help()
            
            elif command in ['add', 'new', 'create']:
                if not args:
                    print("Please specify a task! Example: add 'Buy groceries'")
                    continue
                
                # Check if priority is specified
                task_parts = args.split()
                if len(task_parts) > 1 and task_parts[-1].lower() in ['high', 'medium', 'low']:
                    priority = task_parts[-1].lower()
                    description = ' '.join(task_parts[:-1])
                else:
                    priority = "medium"
                    description = args
                
                task_manager.add_task(description, priority)
            
            elif command in ['list', 'show', 'view', 'ls']:
                show_completed = args.lower() != 'active'
                task_manager.view_tasks(show_completed)
            
            elif command in ['done', 'complete', 'finish']:
                if not args:
                    print("Please specify a task ID! Example: done 5")
                    continue
                task_manager.mark_complete(args)
            
            elif command in ['remove', 'delete', 'rm']:
                if not args:
                    print("Please specify a task ID! Example: remove 3")
                    continue
                task_manager.remove_task(args)
            
            elif command in ['update', 'edit', 'modify']:
                if not args or ' ' not in args:
                    print("Please specify task ID and new description!")
                    print("Example: update 2 'New task description'")
                    continue
                
                try:
                    task_id, new_desc = args.split(' ', 1)
                    task_manager.update_task(task_id, new_desc)
                except ValueError:
                    print("Invalid format. Use: update <id> <new description>")
            
            elif command in ['search', 'find']:
                if not args:
                    print("Please specify a keyword to search for!")
                    continue
                task_manager.search_tasks(args)
            
            else:
                print(f"🤔 I don't recognize '{command}'. Type 'help' to see available commands!")
        
        except KeyboardInterrupt:
            print("\n\n👋 Goodbye! Thanks for using Task Manager!")
            break
        except Exception as e:
            print(f"😅 Oops! Something unexpected happened: {str(e)}")
            print("Don't worry, your tasks are safe! Try again.")

if __name__ == "__main__":
    main()
