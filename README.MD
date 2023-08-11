# Como criar uma API em Django - Criando um CRUD

# Organizando o ambiente

## Criando a venv
    python -m venv venv1

## Ativando a venv
    cd venv1\Scripts
    activate
    cd ../..

## Ativar Interpretador
    crtl + Shift + P 
    python: select interpreter (Recomendado)

## Instalar o Django e Django_rest_framework
    pip install django djangorestframework

## Criar Projeto Django
    django-admin startproject api_root .

## Criação do app
    django-admin startapp api_rest

# Settings.py do projeto
## Adicionar apps instalados em settings.py
    'rest_framework',
    'corsheaders',
    'api_rest'

## Adicionar o Middleware do cosheaders
    'cosheaders.middleware.CorsMiddleware',

## Adicionar endereço do cosheaders no fim do arquivo
    CORS_ALLOW_ORIGINS = [
        'https://localhost:8000'
    ]

# Criando Models

## Criação da model de User
    from django.db import models

    class User(models.Model):
        user_nickname = models.CharField(max_length=100, primary_key=True, default='')
        user_name = models.CharField(max_length=150, default='')
        user_email = models.CharField(default='')
        user_age = models.IntegerField(default=0)

        def __str__(self):
            return f'Nickname: {self.user_nickname} | E-mail: {self.user_email}'

## Criar model no banco de dados
    python manage.py makemigrations
    python manage.py migrate

# Configurando o Admin

## Registrar Model em admin.py
    from django.contrib import admin

    from .models import User

    admin.site.register(User)

## Criar um super usuario
    python manage.py createsuperuser

# Criando Serializers

## Criar arquivo Serializers.py dentro do app e definir o serializer
    from rest_framework import serializers

    from .models import User

    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = '__all__'

# Criando as Views

## Relizar importações para view
    from django.shortcuts import render
    from django.http import HttpResponse, JsonResponse

    from rest_framework.decorators import api_view
    from rest_framework.response import Response
    from rest_framework import status

    from .models import User
    from .serializers import UserSerializer

    import json

# Banco de Dados em Django (Resumo)
## Funções
    def databasEmDjango():
        data = User.objects.get(pk='douglas_nick')      # OBJETO
        
        data = User.objects.filter(user_age='23')       # QUERYSET

        data = User.objects.exclude(user_age='23')      # QUERYSET

        data.save()

        data.delete()

# GET (1/3) - Acessando Todos os Dados (READ)

## Definindo a função GET
    @api_view(['GET'])

    def get_users(request):
        
        if request.method == 'GET':
            users = User.objects.all()
            serializer = UserSerializer(users, many=True)
            return Response(serializer.data)
        
        return Response(status=status.HTTP_400_BAD_REQUEST)

# Definir URLs

## Criar arquivo urls.py na pasta do app e definir url
    from django.contrib import admin
    from django.urls import path, include

    from . import views

    urlpatterns = [
        path('', views.get_users, name='get_all_users'),
    ]

# Incluir url da API no arquivo de URLs do projeto
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('api/', include('api_rest.urls'), name='api_rest_urls'),
    ]

# GET (2/3) - Acessando por Link Direto  (READ)

## Criar uma url para acessar os dados por usuario
    from django.contrib import admin
    from django.urls import path, include

    from . import views

    urlpatterns = [
        path('', views.get_users, name='get_all_users'),
        path('user/<str:nick>', views.get_by_nick)
    ]

## Criar função get_by_nick (Para pegar o usuario pelo nick)
    @api_view(['GET'])
    def get_by_nick(request, nick):

        try:
            user = User.objects.get(pk=nick)
        except:
            return Response(status=status.HTTP_404_NOT_FOUND)
        
        if request.method == 'GET':
            serializer = UserSerializer(user)
            return Response(serializer.data)

# GET (3/3) - Acessando por Parâmetro de URL  (READ)

## Criar url para acessar os dados
    from django.contrib import admin
    from django.urls import path, include

    from . import views

    urlpatterns = [
        path('', views.get_users, name='get_all_users'),
        path('user/<str:nick>', views.get_by_nick),
        path('data/', views.user_manager)
    ]

## Criar função user_manager para pegar o usuario por parâmetros
    # Pega um Usuario especifico pelo parametro passado
    @api_view(['GET', 'POST', 'PUT', 'DELETE'])
    def user_manager(request):
        
        if request.method == 'GET':
            try:
                if request.GET['user']:
                    user_nickname = request.GET['user']
                    try:
                        user = User.objects.get(pk=user_nickname)
                    except:
                        return Response(status=status.HTTP_404_NOT_FOUND)
                    
                    serializer = UserSerializer(user)
                    return Response(serializer.data)
                
                else:
                    return Response(status=status.HTTP_400_BAD_REQUEST)
            
            except:
                return Response(status=status.HTTP_400_BAD_REQUEST)

# POST - Criando Dados (CREATE)

## Definindo a função POST
    if request.method == 'POST':
        
        new_user = request.data
        serializer = UserSerializer(data=new_user)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        return Response(status=status.HTTP_400_BAD_REQUEST)

# PUT (1/2) - Editando dados (EDIT)

## Editando dados pelo REQUEST.DATA
    if request.method == 'PUT':
            
        nickname = request.data['user_nickname']
            
        try:
            update_user = User.objects.get(pk=nickname)
        except:
            return Response(status=status.HTTP_404_NOT_FOUND)    

        serializer = UserSerializer(update_user, data=request.data)

        if serializer.is_valid():
             serializer.save()
            return Response(serializer.data, status=status.HTTP_202_ACCEPTED)
            
    return Response(status=status.HTTP_400_BAD_REQUEST)


# PUT (2/2) - Editando dados (EDIT)

## Editando dados por link direto
    if request.method == 'PUT':
        
        serializer = UserSerializer(user, data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_202_ACCEPTED)
        
        return Response(status=status.HTTP_400_BAD_REQUEST)

# DELETE - Deletando Dados (DELETE)

## Criando função para deletar dados
    if request.method == 'DELETE':

            try:
                user_to_delete = User.objects.get(pk=request.data['user_nickname'])
                user_to_delete.delete()
                return Response(status=status.HTTP_202_ACCEPTED)
            except:
                return Response(status=status.HTTP_400_BAD_REQUEST)
