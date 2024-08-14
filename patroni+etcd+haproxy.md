**Цель:**  

### Создание отказоуйстойчивого кластера на базе Patroni  

********************************

## Будем использовать 8 виртуальных машин  

3 ВМ - Postgresql + patroni  
3 ВМ - etcd   
2 BM - haproxy + keepalived  
*******************************  

### Кол-во задействанный ВМ  

![image](https://github.com/user-attachments/assets/ae8b8b2d-0bea-4e95-9069-eb3fb3fd2a7b)  

### Реализуем данную схему  

![image](https://github.com/user-attachments/assets/35935780-a0a8-454d-86b3-e159668610d7)  

### 1. Ставим кластер из 3-х нод etcd  

![image](https://github.com/user-attachments/assets/148d210e-99ee-4d7e-a932-446e6d0836bb)  

### 2. Patroni  

![image](https://github.com/user-attachments/assets/4cc743ae-2301-47ad-abb9-8eb4c7c092d1)  

Базу дает создать только на мастере, на реплике - ошибка.  

### 3. Делаем switchover (ручное переключение)  

![image](https://github.com/user-attachments/assets/5b1308ab-ff24-4bc7-84c3-f6cd60863e1e)  

### 4. Смотрим через haproxy на web-форме  

![image](https://github.com/user-attachments/assets/9e95103d-a4ed-4f08-92d1-aa87e7cbb557)  

и после смены результат   

![image](https://github.com/user-attachments/assets/ab654424-321c-434d-a934-a1422558deb3)  

Видим ,что лидер поменялся на vm3  

### 5. Также посмотрим на второй ноде haproxy  
было  

![image](https://github.com/user-attachments/assets/cbbb2790-8da9-48a1-8eab-459812b38ccb)  

стало  

![image](https://github.com/user-attachments/assets/9b1db0e0-5b17-4831-ad58-ab76785f33ff)  

### 6. Теперь пробуем автоматическое переключение (failover)  

![image](https://github.com/user-attachments/assets/05adb547-3d03-4375-9a8c-b807caf2a791)  

c vm3 перешло управление на vm2 у кого статус Sync Standby  
ну и смотрим в haproxy  
![image](https://github.com/user-attachments/assets/1234185d-1417-4a37-8afb-155116e517da)    

и на второй ноде (192..159)  

![image](https://github.com/user-attachments/assets/0a1f66a9-a8d8-4126-b03d-4fd37cf667c2)  

### 7. Теперь проверим подключение к нашей бд по виртуальному ip  

![image](https://github.com/user-attachments/assets/9f5ab1bc-fc58-40bb-8278-98b48effcb6e)  

подключение прошло успешно с обоих нод haproxy по виртуальному ip  

### 8. Теперь остановим keepalived службу на haproxy2 и проверим что будет  

![image](https://github.com/user-attachments/assets/9352dd04-8a5b-41ae-b46e-ed9826ed2d1f)  

Мы видим ,что виртуальный ip перешел на haproxy 1-ю ноду.  
Подключение также присутствует  
Зашли в psql по виртуальному ip  

### 9. Посмотрим еще дополнительно кто унас является лидером через curl -утилиту  

![image](https://github.com/user-attachments/assets/83a1291d-c991-455f-a63e-71c845b2ca28)  

Видим ,что лидер у нас vm2 - от него ответ 200 идет на мастер-запрос.  

### 10. Проверим еще, где нам дает создать БД, на какой VM  с Postgresql  

Для теста еще раз переключим мастера на vm1 в ручном режиме.  
Подключимся в psql на 3-х всех нодах(vm1,vm2,vm3)  
И попробуем создать бд  

![image](https://github.com/user-attachments/assets/a611b6ed-7ede-43e0-b5e5-176b6ae0d10e)  

Видим, что дает создать только на мастере (vm1) ,что и логично.  
На репликах выдает ошибку ,т.к. они реплики.  
Еще раз curl посмотрим кто мастер  

![image](https://github.com/user-attachments/assets/181965a9-6f0d-4d2d-bd30-b2201257193d)  

ну и на haproxy в web-интерфейсе  

![image](https://github.com/user-attachments/assets/1953afc8-1761-4c6a-9651-49ffe4834156)  

и на второй ВМ haproxy  

![image](https://github.com/user-attachments/assets/2e2cf047-5e70-4602-95eb-a61e4b9baaf3)  

### 11. Схема реализована.  



