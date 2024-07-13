# Final-module-6
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def survey():
    if request.method == 'POST':
        name = request.form['name']
        age = request.form['age']
        income = request.form['income']
        spending = request.form['spending']

        # You can save the collected data to a database or perform any required analysis here

        return f'Thank you for participating in the survey, {name}!'

    return render_template('survey_form.html')

if __name__ == '__main__':
    app.run(debug=True)

<!DOCTYPE html>
<html>
<head>
    <title>Survey Form</title>
</head>
<body>
    <h2>Participant Survey Form</h2>
    <form method="POST">
        <label for="name">Name:</label><br>
        <input type="text" id="name" name="name"><br><br>

        <label for="age">Age:</label><br>
        <input type="number" id="age" name="age"><br><br>

        <label for="income">Income:</label><br>
        <input type="number" id="income" name="income"><br><br>

        <label for="spending">Spending:</label><br>
        <input type="number" id="spending" name="spending"><br><br>

        <input type="submit" value="Submit">
    </form>
</body>
</html>

from flask import Flask, render_template, request
from pymongo import MongoClient

app = Flask(__name__)

# MongoDB connection
client = MongoClient('mongodb://localhost:27017/')
db = client['survey_db']
collection = db['survey_data']

@app.route('/', methods=['GET', 'POST'])
def survey():
    if request.method == 'POST':
        name = request.form['name']
        age = request.form['age']
        gender = request.form['gender']
        income = request.form['income']
        
        # Get expense categories and corresponding amounts spent
        expenses = {}
        expense_categories = ['utilities', 'entertainment', 'school_fees', 'shopping', 'healthcare']
        for category in expense_categories:
            amount = request.form.get(category)
            if amount:
                expenses[category] = amount
            else:
                expenses[category] = 0
        
        # Insert user data into MongoDB
        user_data = {
            'name': name,
            'age': age,
            'gender': gender,
            'income': income,
            'expenses': expenses
        }
        collection.insert_one(user_data)

        return f'Thank you for participating in the survey, {name}!'

    return render_template('survey_form.html')

if __name__ == '__main__':
    app.run(debug=True)

<!DOCTYPE html>
<html>
<head>
    <title>Survey Form</title>
</head>
<body>
    <h2>Participant Survey Form</h2>
    <form method="POST">
        <label for="name">Name:</label><br>
        <input type="text" id="name" name="name"><br><br>

        <label for="age">Age:</label><br>
        <input type="number" id="age" name="age"><br><br>

        <label for="gender">Gender:</label><br>
        <input type="text" id="gender" name="gender"><br><br>

        <label for="income">Total Income:</label><br>
        <input type="number" id="income" name="income"><br><br>

        <h3>Expenses:</h3>
        <input type="checkbox" id="utilities" name="utilities" value="utilities">
        <label for="utilities">Utilities:</label>
        <input type="number" id="utilities_amount" name="utilities"><br>

        <input type="checkbox" id="entertainment" name="entertainment" value="entertainment">
        <label for="entertainment">Entertainment:</label>
        <input type="number" id="entertainment_amount" name="entertainment"><br>

        <input type="checkbox" id="school_fees" name="school_fees" value="school_fees">

class User:
    def __init__(self, name, age, gender, income, expenses):
        self.name = name
        self.age = age
        self.gender = gender
        self.income = income
        self.expenses = expenses

import csv

# Inside the / route after inserting data into MongoDB
users = collection.find()
file_path = 'survey_data.csv'

with open(file_path, mode='w', newline='') as file:
    fieldnames = ['Name', 'Age', 'Gender', 'Income', 'Utilities', 'Entertainment', 'School Fees', 'Shopping', 'Healthcare']
    writer = csv.DictWriter(file, fieldnames=fieldnames)

    writer.writeheader()
    for user in users:
        user_obj = User(user['name'], user['age'], user['gender'], user['income'], user['expenses'])
        writer.writerow({
            'Name': user_obj.name,
            'Age': user_obj.age,
            'Gender': user_obj.gender,
            'Income': user_obj.income,
            'Utilities': user_obj.expenses.get('utilities', ''),
            'Entertainment': user_obj.expenses.get('entertainment', ''),
            'School Fees': user_obj.expenses.get('school_fees', ''),
            'Shopping': user_obj.expenses.get('shopping', ''),
            'Healthcare': user_obj.expenses.get('healthcare', '')
        })

import pandas as pd

data = pd.read_csv('survey_data.csv')
print(data.head())

import pandas as pd

# Load the CSV file
data = pd.read_csv('survey_data.csv')

# Convert columns to appropriate data types
data['Age'] = data['Age'].astype(int)
data['Income'] = data['Income'].astype(float)

# Display the first few rows of the data
data.head()


import matplotlib.pyplot as plt

# Group data by age and calculate the mean income for each age group
age_income = data.groupby('Age')['Income'].mean().reset_index()

# Plot the age vs. income
plt.figure(figsize=(12, 6))
plt.bar(age_income['Age'], age_income['Income'], color='skyblue')
plt.xlabel('Age')
plt.ylabel('Average Income')
plt.title('Average Income by Age')
plt.grid(axis='y')
plt.show()

import seaborn as sns

# Pivot the data to get  expenses all by gender and spending 
gender_spending = data.melt(id_vars=['Gender'], value_vars=['Utilities', 'Entertainment', 'School Fees', 'Shopping', 'Healthcare'], var_name='Category', value_name='Amount')
gender_spending = gender_spending.groupby(['Gender', 'Category'])['Amount'].sum().reset_index()

# Plot the gender across spending categories
plt.figure(figsize=(12, 8))
sns.barplot(x='Category', y='Amount', hue='Gender', data=gender_spending, palette='Set1')
plt.xlabel('Spending Category')
plt.ylabel('Total Amount Spent')
plt.title('Gender Distribution Across Spending Categories')
plt.xticks(rotation=45)
plt.legend(title='Gender')
plt.tight_layout()
plt.show()


# Save the charts as image files
plt.figure(figsize=(12, 6))
plt.bar(age_income['Age'], age_income['Income'], color='skyblue')
plt.xlabel('Age')
plt.ylabel('Average Income')
plt.title('Average Income by Age')
plt.grid(axis='y')
plt.savefig('average_income_by_age.png')

plt.figure(figsize=(12, 8))
sns.barplot(x='Category', y='Amount', hue='Gender', data=gender_spending, palette='Set1')
plt.xlabel('Spending Category')
plt.ylabel('Total Amount Spent')
plt.title('Gender Distribution Across Spending Categories')
plt.xticks(rotation=45)
plt.legend(title='Gender')
plt.tight_layout()
plt.savefig('gender_distribution_spending_categories.png')

# Host on AWS
 ssh -i -key.pem ubuntu@your-ec2-public-ip
 


