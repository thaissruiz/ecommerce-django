# AULA 1
1)Criar a pasta django_ecommerce : mkdir django_ecommerce
2)Entrar na pasta : cd django_ecommerce
3)Dentro da pasta criar o ambiente virtual:  pipenv --three
4)Ativar o ambiente virtualL : pipenv shell
5)Instalar o Django : pip install django
6)Agora crie uma pasta chamada src e entre nela: mkdir src && cd src
7)Crie um projeto chamado e_commerce: django-admin startproject e_commerce .
Obs. Só pra reforçar, o ponto indica que é para criar o projeto na pasta onde o shell está no momento, no caso a pasta src.
8)Inicie o servidor: python manage.py runserver 
9)Criar git hub remoto 
10)Iniciar git hub

# Aula 2
Criar nova branch:



# Aula 4

Método HTTP usado
O método usado no form foi o POST, pois quero mandar as informações digitadas no formulário para o backend, ou seja, quero salvar no banco.

{% csrf_token %}
O {% csrf_token %} é uma proteção Cross Site Request Forgery, que é um tipo de ataque malicioso a um site, onde comandos não autorizados são transmitidos através de um usuário em quem o site confia.

Diferente do cross-site scripting (XSS), que explora a confiança de um usuário em um site, o CSRF explora a confiança que um site tem no navegador do usuário.

 
Caso não use o {% csrf_token %} no form, na hora que você tentar enviar o formulário, será mostrado a seguinte mensagem de erro:

CSRF verification failed. Request aborted

Utilizando o form da biblioteca do django
Vamos modificar novamente src/templates/contact/view.html, o que mudou tá em laranja no código abaixo.

Onde foi trocado o formulário html normal, hard code, pelo context {{ form }} utilizando a biblioteca forms do django.

E em src/e_commerce/views.py importe ContactForm, instancie um contact form e crie o contact form como variável de contexto.

 

Modifique o arquivo src/e_commerce/forms.py para utilizar widgets personalizados.
Para maiores detalhes sobre Customizing widget instances, acesse:

https://docs.djangoproject.com/en/2.1/ref/forms/widgets/#customizing-widget-instances

Modifique o arquivo src/e_commerce/forms.py para customizar as mensagens de campo obrigatório.

Bora colocar uma  validação de email no src/e_commerce/forms.py só pra testar?

Vamos supor que o email tenha que ser necessariamente do gmail para ser aceito.

  def clean_email(self):
        email = self.cleaned_data.get("email")
        if not "gmail.com" in email:
            raise forms.ValidationError("O Email deve ser do gmail.com")
        return email






# Aula 5:
Modifique o src/e_commerce/forms.py, para criar a classe LoginForm e RegisterForm. E também, faça o import do get_user_model.

1- from django.contrib.auth import get_user_model

2- class LoginForm(forms.Form):
    username = forms.CharField()
    password = forms.CharField(widget=forms.PasswordInput)

class RegisterForm(forms.Form):
    username = forms.CharField()
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    password2 = forms.CharField(label='Confirm password', widget=forms.PasswordInput)

    def clean_username(self):
        username = self.cleaned_data.get('username')
        qs = User.objects.filter(username=username)
        if qs.exists():
            raise forms.ValidationError("Esse usuário já existe, escolha outro nome.")
        return username
    
    def clean_email(self):
        email = self.cleaned_data.get('email')
        qs = User.objects.filter(email=email)
        if qs.exists():
            raise forms.ValidationError("Esse email já existe, tente outro!")
        return email

    def clean(self):
        data = self.cleaned_data
        password = self.cleaned_data.get('password')
        password2 = self.cleaned_data.get('password2')
        if password != password2:
            raise forms.ValidationError("As senhas informadas devem ser iguais!")
        return data



Importe das classes
Agora em src/e_commerce/views.py, importe a classe ContactForm, LoginForm, RegisterForm, importe também authenticate, login, get_user_model e redirect do django, e crie os métodos login_page e register_page.

1- from django.contrib.auth import authenticate, login, get_user_model
2-  redirect from .forms import ContactForm, LoginForm, RegisterForm
3- if request.user.is_authenticated:
        context["premium_content"] = "Você é um usuário Premium"
    return render(request, "home_page.html", context)
4- def login_page(request):
    form = LoginForm(request.POST or None)
    context = {
                    "form": form
              }
    print("User logged in")
    #print(request.user.is_authenticated)
    if form.is_valid():
        print(form.cleaned_data)
        username = form.cleaned_data.get("username")
        password = form.cleaned_data.get("password")
        user = authenticate(request, username=username, password=password) 
        print(user)
        #print(request.user.is_authenticated)
        if user is not None:
            #print(request.user.is_authenticated)
            login(request, user)
            print("Login válido")
            # Redireciona para uma página de sucesso.
            return redirect("/")
        else:
            #Retorna uma mensagem de erro de 'invalid login'.
            print("Login inválido")
    return render(request, "auth/login.html", context)

User = get_user_model()
def register_page(request):
    form = RegisterForm(request.POST or None)
    context = {
                    "form": form
              }
    if form.is_valid():
        print(form.cleaned_data)
        username = form.cleaned_data.get("username")
        email = form.cleaned_data.get("email")
        password = form.cleaned_data.get("password")
        new_user = User.objects.create_user(username, email, password)
        print(new_user)
    return render(request, "auth/register.html", context)

5- get_user_model
6- O get_user_model serve para retornar o model de user ativo no momento.

7- O Django tem duas funções no django.contrib.auth:

authenticate()
login()
authenticate
O authenticate() é para autenticar o username e senha.




8- login
O login() e para logar um usuário. Em uma view ele recebe um objeto HttpRequest e um User.

A função login() salva o ID do usuário na sessão usando o framework de sessão do Django.

Quando você está logando um usuário, você deve chamar authenticate() antes de chamar login().

A função authenticate() seta um atributo sobre o User observando que o backend de autenticação, autenticou o usuário com sucesso

Ele recebe dois argumentos: username e password e retorna um objeto User se a senha for válida para este username e None e a senha for inválida.


9- cleaned_data
O cleaned_data é um dicionário com os dados limpos do formulário, ele retorna esses dados limpos.

Cada campo em uma classe Form é responsável não apenas pela validação de dados, mas também por “limpá-lo”, normalizando-o para um formato consistente.

Esse é um recurso interessante porque permite que os dados de um determinado campo sejam inseridos de várias maneiras, sempre resultando em resultados consistentes.

Por exemplo, DateField normaliza a entrada em um objeto datetime.date do Python, independentemente de você passar uma string no formato ‘2018-07-15’, um objeto datetime.date ou vários outros formatos, DateField sempre o normalizará para um objeto datetime.date, desde que seja válido.

Depois de criar uma instância de formulário com um conjunto de dados e validá-la, você pode acessar os dados limpos por meio do atributo cleaned_data.

10- Agora vamos criar src/template/auth/login.html

11- E o src/template/auth/register.html

12- Vamos fazer as migrações para o banco

python manage.py makemigrations
python manage.py migrate

13- Agora crie um super usuário

python manage.py createsuperuser

14- Escolha um nome e uma senha para esse usuário.

15- Vamos importar login_page, register_page no src/e_commerce/urls.py e adicionar no urlpatterns.

16- Vamos mudar nossa home page para ter um conteúdo premium, para os usuários registrados. Em src/template/home_page.html coloque o conteúdo abaixo.




