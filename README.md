# COUNTRY AND CURRENCY MANAGEMENT APPLICATION USING SQLITE DATABASE

## Register No : 212224220080
## Name : RESHMITHAA B

## AIM

To develop an Android application using Java and SQLite Database to store and manage country names and their respective currencies. The application uses ListView to display records and performs CRUD (Create, Read, Update and Delete) operations.

## ALGORITHM

1. Start the Android application.
2. Create an SQLite database named `CountryDB`.
3. Create a table named `Country` with ID, Country and Currency fields.
4. Initialize the database using `DatabaseHelper.java`.
5. Create database operations using `DBManager.java`.
6. Display all country names and currencies using ListView.
7. Provide an option to add a new country and its currency.
8. Insert the entered details into the SQLite database.
9. Allow the user to select an existing country record.
10. Display the selected record in the modification screen.
11. Allow the user to update the country name and currency.
12. Provide an option to delete the selected record.
13. Refresh the ListView after every database operation.
14. Stop the application.

## SOFTWARE REQUIREMENTS

- Android Studio
- Java Programming Language
- SQLite Database
- Android Emulator
- XML for User Interface Design

## PROGRAM

### 1. DatabaseHelper.java

This class creates and manages the SQLite database and Country table.

```java
package com.example.workshop1;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    public static final String DATABASE_NAME = "CountryDB";
    public static final String TABLE_NAME = "Country";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String query = "CREATE TABLE Country (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "country TEXT, " +
                "currency TEXT)";

        db.execSQL(query);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS Country");
        onCreate(db);
    }
}
```

### 2. DBManager.java

This class performs database operations such as inserting, retrieving, updating and deleting records.

```java
package com.example.workshop1;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class DBManager {

    DatabaseHelper helper;
    SQLiteDatabase database;

    public DBManager(Context context) {
        helper = new DatabaseHelper(context);
    }

    public void open() {
        database = helper.getWritableDatabase();
    }

    public void close() {
        helper.close();
    }

    public long insert(String country, String currency) {
        ContentValues values = new ContentValues();

        values.put("country", country);
        values.put("currency", currency);

        return database.insert("Country", null, values);
    }

    public Cursor getAllCountries() {
        return database.rawQuery("SELECT * FROM Country", null);
    }

    public int update(int id, String country, String currency) {
        ContentValues values = new ContentValues();

        values.put("country", country);
        values.put("currency", currency);

        return database.update(
                "Country",
                values,
                "id=?",
                new String[]{String.valueOf(id)}
        );
    }

    public int delete(int id) {
        return database.delete(
                "Country",
                "id=?",
                new String[]{String.valueOf(id)}
        );
    }
}
```

### 3. CountryListActivity.java

This class displays all country names and currencies using ListView and provides navigation to add and modify screens.

```java
package com.example.workshop1;

import android.app.Activity;
import android.os.Bundle;
import android.content.Intent;
import android.database.Cursor;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ListView;
import java.util.ArrayList;

public class CountryListActivity extends Activity {

    ListView listView;
    Button addButton;
    DBManager dbManager;
    ArrayList<String> countryList;
    ArrayList<Integer> idList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        addButton = findViewById(R.id.addButton);

        dbManager = new DBManager(this);

        addButton.setOnClickListener(v -> {
            Intent intent = new Intent(this, AddCountryActivity.class);
            startActivity(intent);
        });

        listView.setOnItemClickListener((parent, view, position, id) -> {
            Intent intent = new Intent(this, ModifyCountryActivity.class);
            intent.putExtra("id", idList.get(position));
            startActivity(intent);
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadCountries();
    }

    private void loadCountries() {

        countryList = new ArrayList<>();
        idList = new ArrayList<>();

        dbManager.open();

        Cursor cursor = dbManager.getAllCountries();

        if (cursor.moveToFirst()) {
            do {
                int id = cursor.getInt(0);
                String country = cursor.getString(1);
                String currency = cursor.getString(2);

                idList.add(id);
                countryList.add(country + " - " + currency);

            } while (cursor.moveToNext());
        }

        cursor.close();
        dbManager.close();

        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                countryList
        );

        listView.setAdapter(adapter);
    }
}
```

### 4. AddCountryActivity.java

This class accepts country name and currency as input and inserts the details into the SQLite database.

```java
package com.example.workshop1;

import android.app.Activity;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

public class AddCountryActivity extends Activity {

    EditText countryEdit, currencyEdit;
    Button saveButton;
    DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_country);

        countryEdit = findViewById(R.id.countryEdit);
        currencyEdit = findViewById(R.id.currencyEdit);
        saveButton = findViewById(R.id.saveButton);

        dbManager = new DBManager(this);

        saveButton.setOnClickListener(v -> {

            String country = countryEdit.getText().toString().trim();
            String currency = currencyEdit.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this, "Enter all details", Toast.LENGTH_SHORT).show();
                return;
            }

            dbManager.open();
            long result = dbManager.insert(country, currency);
            dbManager.close();

            if (result != -1) {
                Toast.makeText(this, "Country Added Successfully", Toast.LENGTH_SHORT).show();
                finish();
            } else {
                Toast.makeText(this, "Failed to Add Country", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

### 5. ModifyCountryActivity.java

This class allows users to view, modify and delete an existing country record.

```java
package com.example.workshop1;

import android.app.Activity;
import android.os.Bundle;
import android.database.Cursor;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

public class ModifyCountryActivity extends Activity {

    EditText countryEdit, currencyEdit;
    Button updateButton, deleteButton;
    DBManager dbManager;
    int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_country);

        countryEdit = findViewById(R.id.countryEdit);
        currencyEdit = findViewById(R.id.currencyEdit);
        updateButton = findViewById(R.id.updateButton);
        deleteButton = findViewById(R.id.deleteButton);

        dbManager = new DBManager(this);

        countryId = getIntent().getIntExtra("id", -1);

        loadCountry();

        updateButton.setOnClickListener(v -> {

            String country = countryEdit.getText().toString().trim();
            String currency = currencyEdit.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this, "Enter all details", Toast.LENGTH_SHORT).show();
                return;
            }

            dbManager.open();
            int result = dbManager.update(countryId, country, currency);
            dbManager.close();

            if (result > 0) {
                Toast.makeText(this, "Updated Successfully", Toast.LENGTH_SHORT).show();
                finish();
            }
        });

        deleteButton.setOnClickListener(v -> {

            dbManager.open();
            int result = dbManager.delete(countryId);
            dbManager.close();

            if (result > 0) {
                Toast.makeText(this, "Deleted Successfully", Toast.LENGTH_SHORT).show();
                finish();
            }
        });
    }

    private void loadCountry() {

        dbManager.open();
        Cursor cursor = dbManager.getAllCountries();

        if (cursor.moveToFirst()) {
            do {
                int id = cursor.getInt(0);

                if (id == countryId) {
                    countryEdit.setText(cursor.getString(1));
                    currencyEdit.setText(cursor.getString(2));
                    break;
                }

            } while (cursor.moveToNext());
        }

        cursor.close();
        dbManager.close();
    }
}
```

### 6. MainActivity.java

This class launches the CountryListActivity when the application starts.

```java
package com.example.workshop1;

import android.content.Intent;
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent(MainActivity.this, CountryListActivity.class);
        startActivity(intent);
        finish();
    }
}
```

## XML LAYOUT FILES

### 7. activity_country_list.xml

This layout displays the country list and Add Country button.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Country and Currency"
        android:textSize="24sp"
        android:textStyle="bold"
        android:gravity="center"
        android:padding="16dp" />

    <Button
        android:id="@+id/addButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Country" />

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

### 8. activity_add_country.xml

This layout provides input fields to enter country name and currency.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Country"
        android:textSize="24sp"
        android:textStyle="bold"
        android:gravity="center"
        android:padding="16dp" />

    <EditText
        android:id="@+id/countryEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Country Name" />

    <EditText
        android:id="@+id/currencyEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Currency" />

    <Button
        android:id="@+id/saveButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Save Country" />

</LinearLayout>
```

### 9. activity_modify_country.xml

This layout provides input fields and buttons to update or delete a country record.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Modify Country"
        android:textSize="24sp"
        android:textStyle="bold"
        android:gravity="center"
        android:padding="16dp" />

    <EditText
        android:id="@+id/countryEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Country Name" />

    <EditText
        android:id="@+id/currencyEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Currency" />

    <Button
        android:id="@+id/updateButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update" />

    <Button
        android:id="@+id/deleteButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Delete" />

</LinearLayout>
```

### 10. AndroidManifest.xml

Register the application activities. MainActivity is configured as the launcher activity.

Add these declarations inside the application tag:

```xml
<activity
    android:name=".ModifyCountryActivity"
    android:exported="false" />

<activity
    android:name=".AddCountryActivity"
    android:exported="false" />

<activity
    android:name=".CountryListActivity"
    android:exported="false" />

<activity
    android:name=".MainActivity"
    android:exported="true">

    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

</activity>
```

## OUTPUT

<img width="1366" height="768" alt="Screenshot (468)" src="https://github.com/user-attachments/assets/8f9961d9-5614-4aad-b817-09ff4aa96325" />

<img width="1366" height="768" alt="Screenshot (470)" src="https://github.com/user-attachments/assets/e337a5f6-84ce-46e3-a196-ba15823b396e" />

<img width="1366" height="768" alt="Screenshot (470)" src="https://github.com/user-attachments/assets/1b3480a5-5e7a-4bdc-b3b8-9141c22aba9c" />


## RESULT

Thus, the Android application for managing country names and their respective currencies was developed using Java, SQLite Database and ListView. The application implements all four CRUD operations: Create, Read, Update and Delete successfully.
