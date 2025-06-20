from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3

app = Flask(__name__)
import secrets
app.secret_key = secrets.token_hex(16) 

# Route to render the homepage
@app.route('/', methods=['GET'])
def home():
    query = request.args.get('query')  # Get search query from the form
    conn = sqlite3.connect('services.db')
    c = conn.cursor()

    if query:
        # Search the service provider details
        c.execute('''SELECT name, service_name, area, phone, amount, comment, booking_option FROM service_providers 
                     WHERE name LIKE ? OR service_name LIKE ? OR area LIKE ?''', 
                     ('%' + query + '%', '%' + query + '%', '%' + query + '%'))
    else:
        # Show all service providers if no search query
        c.execute('SELECT name, service_name, area, phone, amount, comment, booking_option FROM service_providers')

    providers = c.fetchall()
    conn.close()
    return render_template('home.html', providers=providers)

# Route to filter services based on the selected field
@app.route('/filter/<service>')
def filter_service(service):
    conn = sqlite3.connect('services.db')
    c = conn.cursor()
    query = 'SELECT name, service_name, area, phone, amount, comment, booking_option FROM service_providers WHERE field = ?'
    c.execute(query, (service,))
    filtered_providers = c.fetchall()
    conn.close()
    return render_template('home.html', providers=filtered_providers)

# Route to render the registration form
@app.route('/register')
def register_form():
    return render_template('register.html')


@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        # Get form data
        name = request.form.get('name')
        service_name = request.form.get('service_name')
        area = request.form.get('area')
        phone = request.form.get('phone')
        amount = request.form.get('amount')
        comment = request.form.get('comment')
        booking_option = request.form.get('booking_option')
        field = request.form.get('field')

         # Open a connection to the database
        conn = sqlite3.connect('services.db')
        c = conn.cursor()

        # Insert data into the service_providers table
        query = '''INSERT INTO service_providers (name, service_name, area, phone, amount, comment, booking_option, field)
               VALUES (?, ?, ?, ?, ?, ?, ?, ?)'''
        c.execute(query, (name, service_name, area, phone, amount, comment, booking_option, field))

        conn.commit()
        conn.close()
        flash('Successfully submitted! You will be redirected to the home page in 3 seconds.')
        return redirect(url_for('register', success=True))

    return render_template('register.html')

@app.route('/search')
def search():
    query = request.args.get('query', '').lower()  # Get the search query, default to empty string if None

    # Connect to the database
    conn = sqlite3.connect('services.db')
    cursor = conn.cursor()

    # Search query to match providers by name, service, area, or phone
    cursor.execute("""
        SELECT name, service_name, area, phone, amount, comment, booking_option
        FROM service_providers
        WHERE lower(name) LIKE ? OR lower(service_name) LIKE ? OR lower(area) LIKE ? OR lower(phone) LIKE ?
    """, (f"%{query}%", f"%{query}%", f"%{query}%", f"%{query}%"))

    filtered_providers = cursor.fetchall()

    # Close the connection
    conn.close()

    # Render the template with the filtered providers
    return render_template('home.html', providers=filtered_providers)

if __name__ == '__main__':
    app.run(debug=True)
