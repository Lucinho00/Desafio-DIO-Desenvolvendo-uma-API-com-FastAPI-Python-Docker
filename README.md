# Desafio-DIO-Desenvolvendo-uma-API-com-FastAPI-Python-Docker

Código
Definição das Rotas:

Em app/routes/, crie um arquivo category_routes.py para as rotas relacionadas às Categorias:
from flask import Blueprint, jsonify, request
from app import db
from app.models.category import Category
from app.schemas.category_schema import CategorySchema

category_bp = Blueprint('category', __name__, url_prefix='/api/category')
category_schema = CategorySchema()
categories_schema = CategorySchema(many=True)

@category_bp.route('/', methods=['GET'])
def get_categories():
    categories = Category.query.all()
    return jsonify(categories_schema.dump(categories))

@category_bp.route('/', methods=['POST'])
def create_category():
    new_category = Category(
        name=request.json['name']
    )
    db.session.add(new_category)
    db.session.commit()
    return category_schema.jsonify(new_category), 201

# Adicione outras rotas conforme necessário (GET, PUT, DELETE)

Registrar as Blueprints:

Em app/__init__.py, registre as blueprints das rotas:
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from config import SQLALCHEMY_DATABASE_URI, SQLALCHEMY_TRACK_MODIFICATIONS

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = SQLALCHEMY_TRACK_MODIFICATIONS

db = SQLAlchemy(app)

from app.routes.category_routes import category_bp
app.register_blueprint(category_bp)

# Adicione outros blueprints conforme necessário

Passo 4: Configuração do Alembic para Migrações
Configuração do Alembic:

Dentro da pasta alembic/, configure alembic.ini para apontar para o banco de dados:
sqlalchemy.url = sqlite:///app.db
Criação das Migrações:

Execute os seguintes comandos para inicializar o ambiente do Alembic e criar a primeira migração:
alembic init alembic
alembic revision -m "initial"
Isso criará os diretórios e arquivos necessários dentro de alembic/ para gerenciar as migrações do banco de dados.
Passo 5: Docker Compose
Configuração do Docker Compose:

Crie um arquivo docker-compose.yml na raiz do projeto para configurar os serviços Docker, incluindo a aplicação Flask e o banco de dados.
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    depends_on:
      - db
  db:
    image: "postgres:12"
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: example
Aqui configuramos um serviço web para construir a imagem a partir do Dockerfile na raiz e um serviço db usando PostgreSQL como exemplo.
Dockerfile:

Crie um arquivo Dockerfile na raiz do projeto para configurar a imagem da aplicação Flask:
FROM python:3.9

WORKDIR /code

COPY requirements.txt /code/
RUN pip install -r requirements.txt

COPY . /code/

CMD ["python", "run.py"]
Inicialização da Aplicação:

Crie um arquivo run.py na raiz do projeto para inicializar a aplicação Flask:
from app import app, db
from app.models import *

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True, host='0.0.0.0')
Passo 6: Executando a Aplicação
Construa e inicie os serviços Docker:

Na raiz do projeto, execute:
docker-compose up --build
Isso vai construir as imagens e iniciar os serviços definidos no docker-compose.yml.
Testando a API com Postman:

Use o Postman para testar as rotas definidas. Por exemplo, GET /api/category para listar as categorias, POST /api/category para criar uma nova categoria, etc.
Observações:
Este é um exemplo básico para ilustrar a estrutura e configuração inicial de uma aplicação Flask com SQLAlchemy, Alembic e Docker. Dependendo das necessidades do projeto, poderá ser adicionar mais funcionalidades, como autenticação, validação de entrada, tratamento de erros, entre outros.
