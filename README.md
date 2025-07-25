# flask.py
from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)
DATABASE = 'inventory.db'

def init_db():
    with sqlite3.connect(DATABASE) as conn:
        conn.execute('''
            CREATE TABLE IF NOT EXISTS items (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                quantity INTEGER NOT NULL,
                price REAL NOT NULL
            )
        ''')

@app.route('/')
def index():
    with sqlite3.connect(DATABASE) as conn:
        cursor = conn.execute('SELECT * FROM items')
        items = cursor.fetchall()
    return render_template('index.html', items=items)

@app.route('/create', methods=['GET', 'POST'])
def create():
    if request.method == 'POST':
        name = request.form['name']
        quantity = request.form['quantity']
        price = request.form['price']
        with sqlite3.connect(DATABASE) as conn:
            conn.execute('INSERT INTO items (name, quantity, price) VALUES (?, ?, ?)',
                         (name, quantity, price))
        return redirect(url_for('index'))
    return render_template('create.html')

@app.route('/update/<int:id>', methods=['GET', 'POST'])
def update(id):
    if request.method == 'POST':
        name = request.form['name']
        quantity = request.form['quantity']
        price = request.form['price']
        with sqlite3.connect(DATABASE) as conn:
            conn.execute('UPDATE items SET name=?, quantity=?, price=? WHERE id=?',
                         (name, quantity, price, id))
        return redirect(url_for('index'))

    with sqlite3.connect(DATABASE) as conn:
        cursor = conn.execute('SELECT * FROM items WHERE id=?', (id,))
        item = cursor.fetchone()
    return render_template('update.html', item=item)

@app.route('/delete/<int:id>')
def delete(id):
    with sqlite3.connect(DATABASE) as conn:
        conn.execute('DELETE FROM items WHERE id=?', (id,))
    return redirect(url_for('index'))

if __name__ == '__main__':
    init_db()
    app.run(debug=True)

flask.html
<!DOCTYPE html>
<html>
<head>
    <title>Inventory System</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="container mt-4">
    <h1>Inventory</h1>
    <a href="/create" class="btn btn-primary mb-3">Add Item</a>
    <table class="table table-bordered">
        <thead><tr><th>ID</th><th>Name</th><th>Qty</th><th>Price</th><th>Actions</th></tr></thead>
        <tbody>
            {% for item in items %}
            <tr>
                <td>{{ item[0] }}</td>
                <td>{{ item[1] }}</td>
                <td>{{ item[2] }}</td>
                <td>${{ item[3] }}</td>
                <td>
                    <a href="/update/{{ item[0] }}" class="btn btn-sm btn-warning">Edit</a>
                    <a href="/delete/{{ item[0] }}" class="btn btn-sm btn-danger">Delete</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>

create.html

<!DOCTYPE html>
<html>
<head>
    <title>Add Item</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="container mt-4">
    <h2>Add New Item</h2>
    <form method="post">
        <div class="mb-3">
            <label>Name:</label>
            <input type="text" name="name" class="form-control" required>
        </div>
        <div class="mb-3">
            <label>Quantity:</label>
            <input type="number" name="quantity" class="form-control" required>
        </div>
        <div class="mb-3">
            <label>Price:</label>
            <input type="number" step="0.01" name="price" class="form-control" required>
        </div>
        <button type="submit" class="btn btn-success">Add</button>
        <a href="/" class="btn btn-secondary">Cancel</a>
    </form>
</body>
</html>
update.html

<!DOCTYPE html>
<html>
<head>
    <title>Update Item</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="container mt-4">
    <h2>Edit Item</h2>
    <form method="post">
        <div class="mb-3">
            <label>Name:</label>
            <input type="text" name="name" class="form-control" value="{{ item[1] }}" required>
        </div>
        <div class="mb-3">
            <label>Quantity:</label>
            <input type="number" name="quantity" class="form-control" value="{{ item[2] }}" required>
        </div>
        <div class="mb-3">
            <label>Price:</label>
            <input type="number" step="0.01" name="price" class="form-control" value="{{ item[3] }}" required>
        </div>
        <button type="submit" class="btn btn-primary">Update</button>
        <a href="/" class="btn btn-secondary">Cancel</a>
    </form>
</body>
</html>
