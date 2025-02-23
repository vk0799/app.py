from flask import Flask, render_template, request, redirect, url_for, session
from werkzeug.security import generate_password_hash, check_password_hash
from functools import wraps

app = Flask(__name__)  # __name__ ಬಳಸಿಕೊಳ್ಳಿ
app.secret_key = 'your_secret_key'

# Sample user data
users = {
    "admin": generate_password_hash("password")
}

# Sample fertilizer data
fertilizers = [
    {"id": 1, "name": "Organic Compost", "price": 500, "image": "compost.jpg"},
    {"id": 2, "name": "Nitrogen Fertilizer", "price": 700, "image": "nitrogen.jpg"},
    {"id": 3, "name": "Phosphorus Fertilizer", "price": 650, "image": "phosphorus.jpg"},
    {"id": 4, "name": "Potassium Fertilizer", "price": 800, "image": "potassium.jpg"}
]

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/')
def home():
    return render_template('index.html', fertilizers=fertilizers, user=session.get('user'))

@app.route('/product/<int:fertilizer_id>')
def product(fertilizer_id):
    fertilizer = next((f for f in fertilizers if f["id"] == fertilizer_id), None)
    return render_template('product.html', fertilizer=fertilizer, user=session.get('user'))

@app.route('/cart')
@login_required
def cart():
    return render_template('cart.html', user=session.get('user'))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username in users and check_password_hash(users[username], password):
            session['user'] = username
            return redirect(url_for('home'))
        else:
            return "Invalid credentials!", 401
    return render_template('login.html')

@app.route('/logout')
def logout():
    session.pop('user', None)
    return redirect(url_for('home'))

if __name__ == '__main__':  # __name__ ಬಳಸಿಕೊಳ್ಳಿ
    app.run(debug=True)
