Craft&Co. - Ready-to-paste E-commerce Backend (FastAPI)

Features: Product CRUD, Orders, JWT Auth Admin Panel (separate), MongoDB, UPI Payment Integration

from fastapi import FastAPI, HTTPException, Depends from fastapi.middleware.cors import CORSMiddleware from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm from pydantic import BaseModel from passlib.context import CryptContext from jose import jwt, JWTError from pymongo import MongoClient from bson import ObjectId import os

---------------- CONFIG ----------------

SECRET_KEY = "dffb7841e4894a2cbcf53bc0cf61d59ac5ec1fa5e625dd1e1c"  # secure secret ALGORITHM = "HS256" ADMIN_EMAIL = "rayhanshaikh026@gmail.com" ADMIN_PASSWORD_HASH = "$2b$12$ZpWOfzGzHz.1GpM25X28eu5I9bTD9sVAJuFlVGUQNmEsYHtqF/54i"  # hashed secure password MONGO_URI = os.getenv("MONGO_URI", "mongodb+srv://<username>:<password>@<cluster-url>/craftco?retryWrites=true&w=majority")

---------------- INIT ----------------

app = FastAPI() app.add_middleware( CORSMiddleware, allow_origins=[""], allow_credentials=True, allow_methods=[""], allow_headers=["*"], ) client = MongoClient(MONGO_URI) db = client["craftco"] products_col = db["products"] orders_col = db["orders"] oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/admin/login") pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

---------------- UTILS ----------------

def verify_password(plain, hashed): return pwd_context.verify(plain, hashed)

def create_token(data: dict): return jwt.encode(data, SECRET_KEY, algorithm=ALGORITHM)

def get_current_user(token: str = Depends(oauth2_scheme)): try: payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM]) email = payload.get("sub") if email != ADMIN_EMAIL: raise HTTPException(status_code=401, detail="Invalid credentials") return email except JWTError: raise HTTPException(status_code=401, detail="Invalid token")

---------------- MODELS ----------------

class Product(BaseModel): title: str description: str price: float category: str image_url: str

class Order(BaseModel): customer_name: str customer_phone: str customer_address: str items: list total_amount: float payment_status: str  # Paid/Pending

---------------- ROUTES ----------------

@app.post("/api/admin/login") async def admin_login(form: OAuth2PasswordRequestForm = Depends()): if form.username != ADMIN_EMAIL or not verify_password(form.password, ADMIN_PASSWORD_HASH): raise HTTPException(status_code=400, detail="Incorrect email or password") token = create_token({"sub": ADMIN_EMAIL}) return {"access_token": token, "token_type": "bearer"}

@app.post("/api/products", dependencies=[Depends(get_current_user)]) async def add_product(product: Product): products_col.insert_one(product.dict()) return {"message": "Product added"}

@app.get("/api/products") async def get_products(): products = [] for p in products_col.find(): p["_id"] = str(p["_id"]) products.append(p) return products

@app.delete("/api/products/{product_id}", dependencies=[Depends(get_current_user)]) async def delete_product(product_id: str): products_col.delete_one({"_id": ObjectId(product_id)}) return {"message": "Product deleted"}

@app.post("/api/orders") async def place_order(order: Order): orders_col.insert_one(order.dict()) return {"message": "Order placed. Please complete UPI payment at mehbub760-1@okhdfcbank."}

@app.get("/api/orders", dependencies=[Depends(get_current_user)]) async def get_orders(): orders = [] for o in orders_col.find(): o["_id"] = str(o["_id"]) orders.append(o) return orders

---------------- SAMPLE DATA LOADER ----------------

@app.post("/api/admin/load-sample-products", dependencies=[Depends(get_current_user)]) async def load_sample_products(): sample_products = [ {"title": "Premium Black T-Shirt", "description": "Crafted from soft cotton.", "price": 1499, "category": "T-Shirts", "image_url": "https://i.imgur.com/sample1.jpg"}, {"title": "Classic Grey Hoodie", "description": "Warm and comfortable.", "price": 2499, "category": "Hoodies", "image_url": "https://i.imgur.com/sample2.jpg"}, {"title": "White Formal Shirt", "description": "Tailored for perfection.", "price": 1999, "category": "Shirts", "image_url": "https://i.imgur.com/sample3.jpg"}, {"title": "Black Premium Sneakers", "description": "Elegant and durable.", "price": 3999, "category": "Sneakers", "image_url": "https://i.imgur.com/sample4.jpg"}, {"title": "Slim Fit Joggers", "description": "Comfort with style.", "price": 1799, "category": "Lowers", "image_url": "https://i.imgur.com/sample5.jpg"}, ] products_col.insert_many(sample_products) return {"message": "Sample products loaded"}

---------------- END ----------------

Instructions:

1. Paste this code in Railway or Render as a FastAPI project.

2. Set your MONGO_URI in environment variables.

3. Deploy and your backend with admin panel and product management is ready.

4. Connect this API to your Next.js frontend for Craft&Co.

5. Secure and ready to receive orders with payment instructions to your UPI.

# Craft-Co.
