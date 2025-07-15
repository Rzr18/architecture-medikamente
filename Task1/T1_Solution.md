Для повышения безопасности данных текущую ситуацию можно значительно улучшить без значительных вложений, выполнив следующие шаги:
1. разделить данные по типу и чуствительности, 
2. применить шифрование на файловом сервере для данных в покое (AES256)
3. настроить разграничение доступа для разных типов данных
4. настроить шифрование при передаче (SMB пока офис один, TLS - при открытии новых филиалов)
5. использовать псевдоним клиента (номер) в медицинских журналах

## Регистрация

```mermaid
flowchart TD    
    client[Клиент]
    resep[Ресепшен]

    pr_register(Регистрация клиента)
    
    D1[[Файл учета пациентов <br> PII:base <br> **Шифрованние AES256 <br> RBAC: Ресепшен, Доктора** ]]
    D2[[Файлы документов <br> PII:sens <br> **Шифрованние AES256 <br> RBAC: Только ресепшен**]]

    client -->|"Предоставляет документы <br> PII:sens, PII:base"| resep 
    resep --> pr_register
    pr_register -->|Записывает общие данные <br> в список пациентов <br> формирует псевдоним <br> TLS| D1
    pr_register -->|Записывает персональные <br>данные в файл пациента <br>сохраняет сканы <br> TLS| D2

    classDef process fill:#7ac3f0
    classDef data fill:#eee3e0
    class D1,D2,D3 data
    class pr_register process
```

## Запись на прием

```mermaid
flowchart TD    
    client[Клиент]
    resep[Ресепшен]

    pr(Запись на прием)
    
    DP[[Журнал приема <br> PHI <br> **Шифрованние AES256 <br> RBAC: Ресепшен, Доктора** ]]
    
    client -->|"Сообщает ФИО <br> время, специалист <br> PII:base"| resep 
    resep -->|Получение псевдонима| pr
    pr -->|Карточка приема <br> ФИО + псевдоним| client
    pr -->|Записывает в <br> журнал приемов без PII <br> PHI <br> TLS| DP
    
    classDef process fill:#7ac3f0
    classDef data fill:#eee3e0
    class D1,D2,D3 data
    class pr process
```

## Прием пациента

```mermaid
flowchart TD    
    client[Клиент]
    doc[Специалист]

    pra(Проведение анализов)
    prp(Прием)
    
    DP[[Журнал приема <br> PHI <br> **Шифрованние AES256 <br> RBAC: Ресепшен, Доктора** ]]
    DA[[Журнал анализов <br> PHI <br> **Шифрованние AES256 <br> RBAC: Доктора** ]]
    
    client -->|"Карточка приема <br> с псевдонимом <br> Симптомы <br> PII:base, PHI"| doc 
    doc --> prp
    prp --> pra
    prp -->|Заполняет журнал приема <br> Диагноз, Назначение <br> PHI <br> TLS| DP
    pra -->|Заполняет журнал анализов <br> Анализ, Заключение <br> PHI <br> TLS| DA
    
    classDef process fill:#7ac3f0
    classDef data fill:#eee3e0
    class D1,D2,D3 data
    class pra,prp process
```
## Оплата услуг
```mermaid
flowchart TD    
    client[Клиент]
    pers[Кассир]
    kkm[ККМ]

    pr(Проведение оплаты)
    
    D1[[Журнал оплат <br> FI <br> **Шифрованние AES256 <br> RBAC: Кассир** ]]
    D2[[1С Бухгалтерия> <br> FI <br> **Шифрованние AES256 <br> RBAC: Кассир, Бухгалтер** ]]
    
    client -->|"Карточка приема или ФИО <br> PII:base"| pers 
    pers -->|Получает псевдоним <br> из журнала приема| pr
    pr --> kkm
    pr -->|Заполняет журнал оплат <br> FI <br> TLS| D1
    pr -->|Проводит в 1С <br> Анализ, Заключение <br> PII:base, FI <br> TLS/SSL| D2
    kkm -->|TCP/IP OLE| D2
    
    classDef process fill:#7ac3f0
    classDef data fill:#eee3e0
    class D1,D2,D3 data
    class pr,prp process
```
## Склад
```mermaid
flowchart TD    
    pers[Кладовщик]

    pr(Учет ТМЦ)
    
    D[[1С Торговля и склад <br> **STI <br> RBAC: Кладовщик** ]]
    
    pers -->|Прием, выдача со склада| pr
    pr -->|Проводит в 1С изменения <br> STI <br> TLS/SSL| D
    
    
    classDef process fill:#7ac3f0
    classDef data fill:#eee3e0
    class D data
    class pr,prp process
```

