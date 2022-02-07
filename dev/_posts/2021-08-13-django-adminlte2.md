# Django adminlte2 template

- 목적 : 온보딩을 위해 시스템 관리 페이지 생성


## 설치 과정 설치
- django project 생성
- adminlte2 설치
- settings.py에 INSTALLED_APP 추가
```
'django_adminlte',
'django_adminlte_theme',
```

- STATIC_ROOT 변수 추가
```
STATIC_ROOT = (os.path.join('static'))
```

- run command
```
python manage.py collectstatic
python manage.py migrate
python manage.py makemigrations
python manage.py createsuperuser
```
