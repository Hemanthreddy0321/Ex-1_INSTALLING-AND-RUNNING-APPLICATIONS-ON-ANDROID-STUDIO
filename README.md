# Ex-1_INSTALLING-AND-RUNNING-APPLICATIONS-ON-ANDROID-STUDIO

# INSTALLING AND RUNNING APPLICATIONS ON ANDROID STUDIO

**Name:** Hemanth A
**Register Number:** 212223220035
**Department:** B.Tech Information Technology  

## 1. AIM

To develop an Android application that allows users to add, view, mark as completed, and delete daily tasks.

## 2. SOFTWARE REQUIREMENTS

1. Operating System: Windows 10/11
2. IDE: Android Studio
3. Programming Language: Java
4. Frontend: XML
5. Storage: SharedPreferences
6. Build Tool: Gradle
7. Android SDK: API Level 23 or above
8. Emulator: Android Virtual Device (AVD)
9. JDK: Java Development Kit

## 3. ALGORITHM

1. Start the application.
2. Initialize the Android activity.
3. Create the user interface using XML.
4. Enter the task in the input field.
5. Click the Add Task button.
6. Validate the entered task.
7. Add the task to the task list.
8. Display all tasks on the screen.
9. Allow the user to mark tasks as completed using CheckBox.
10. Allow the user to delete tasks.
11. Save task details using SharedPreferences.
12. Load previously saved tasks when the application is reopened.
13. Stop the application.

## 4. PROGRAM

### A. XML Code – `activity_main.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp"
    android:background="#F5F6FA">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="My To-Do List"
        android:textSize="28sp"
        android:textStyle="bold"
        android:textColor="#202124"
        android:gravity="center"
        android:padding="15dp"/>

    <EditText
        android:id="@+id/taskInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter your task"
        android:padding="12dp"
        android:singleLine="true"
        android:inputType="textCapSentences"
        android:backgroundTint="#6750A4"/>

    <Button
        android:id="@+id/addButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Task"
        android:layout_marginTop="10dp"/>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Your Tasks"
        android:textSize="20sp"
        android:textStyle="bold"
        android:textColor="#202124"
        android:paddingTop="20dp"
        android:paddingBottom="10dp"/>

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <LinearLayout
            android:id="@+id/taskContainer"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"/>

    </ScrollView>

</LinearLayout>
```
### B. Java Code – `MainActivity.java`
```

package com.example.todomylist;

import android.os.Bundle;
import android.graphics.Color;
import android.view.Gravity;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.Toast;
import android.content.SharedPreferences;

import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONArray;
import org.json.JSONObject;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText taskInput;
    Button addButton;
    LinearLayout taskContainer;

    ArrayList<Task> taskList = new ArrayList<>();

    SharedPreferences preferences;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        taskInput = findViewById(R.id.taskInput);
        addButton = findViewById(R.id.addButton);
        taskContainer = findViewById(R.id.taskContainer);

        preferences = getSharedPreferences(
                "TodoData", MODE_PRIVATE);

        loadTasks();
        displayTasks();

        addButton.setOnClickListener(v -> {

            String taskName =
                    taskInput.getText().toString().trim();

            if (taskName.isEmpty()) {
                Toast.makeText(this,
                        "Please enter a task",
                        Toast.LENGTH_SHORT).show();
            } else {

                taskList.add(new Task(taskName, false));

                taskInput.setText("");

                saveTasks();
                displayTasks();

                Toast.makeText(this,
                        "Task Added Successfully",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }

    // Display all tasks
    private void displayTasks() {

        taskContainer.removeAllViews();

        for (int i = 0; i < taskList.size(); i++) {

            Task task = taskList.get(i);

            LinearLayout row = new LinearLayout(this);

            row.setOrientation(LinearLayout.HORIZONTAL);
            row.setGravity(Gravity.CENTER_VERTICAL);
            row.setPadding(8, 8, 8, 8);

            CheckBox checkBox = new CheckBox(this);

            checkBox.setText(task.name);
            checkBox.setTextSize(16);
            checkBox.setTextColor(Color.BLACK);
            checkBox.setChecked(task.completed);

            LinearLayout.LayoutParams checkParams =
                    new LinearLayout.LayoutParams(
                            0,
                            LinearLayout.LayoutParams.WRAP_CONTENT,
                            1
                    );

            row.addView(checkBox, checkParams);

            // Mark task as completed
            checkBox.setOnCheckedChangeListener(
                    (buttonView, isChecked) -> {

                        task.completed = isChecked;

                        saveTasks();
                    });

            Button deleteButton = new Button(this);

            deleteButton.setText("Delete");
            deleteButton.setTextSize(12);

            row.addView(deleteButton);

            // Delete task
            deleteButton.setOnClickListener(v -> {

                taskList.remove(task);

                saveTasks();
                displayTasks();

                Toast.makeText(this,
                        "Task Deleted",
                        Toast.LENGTH_SHORT).show();
            });

            taskContainer.addView(row);
        }
    }

    // Save tasks
    private void saveTasks() {

        JSONArray jsonArray = new JSONArray();

        try {

            for (Task task : taskList) {

                JSONObject object = new JSONObject();

                object.put("name", task.name);
                object.put("completed", task.completed);

                jsonArray.put(object);
            }

            preferences.edit()
                    .putString("tasks", jsonArray.toString())
                    .apply();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Load saved tasks
    private void loadTasks() {

        String savedData =
                preferences.getString("tasks", "[]");

        try {

            JSONArray jsonArray = new JSONArray(savedData);
            taskList.clear();

            for (int i = 0; i < jsonArray.length(); i++) {

                JSONObject object = jsonArray.getJSONObject(i);

                String name = object.getString("name");

                boolean completed =
                        object.getBoolean("completed");

                taskList.add(new Task(name, completed));
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Task model
    static class Task {

        String name;
        boolean completed;

        Task(String name, boolean completed) {

            this.name = name;
            this.completed = completed;
        }
    }
}

```
                
## 5. PROCEDURE

1. Open Android Studio and create a new project.
2. Select Empty Views Activity.
3. Enter the project name as ToDoList.
4. Select Java as the programming language.
5. Design the user interface using XML.
6. Add an EditText to enter tasks.
7. Add a Button to insert tasks.
8. Add a LinearLayout to display the task list.
9. Write Java code to handle adding, completing and deleting tasks.
10. Implement SharedPreferences to save task details.
11. Implement JSON to store and retrieve task information.
12. Build and run the application using an Android emulator.
13. Test all the application functionalities.

## 6. OUTPUT :
<img width="1917" height="976" alt="image" src="https://github.com/user-attachments/assets/26d6ae89-46fd-477a-a777-68cb581575d0" />


## 7. RESULT

Thus, the To-Do List Android application was successfully developed using Java and XML in Android Studio. The application allows users to add, view, mark tasks as completed, delete tasks, and save task details using SharedPreferences.
