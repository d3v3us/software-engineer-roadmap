# Анализ вопросов из razeone/backend-interview-questions

## Источник
https://github.com/razeone/backend-interview-questions

## Категории вопросов

### 1. Basic Questions (Data Structures, Algorithms and General Programming)

#### ✅ Уже покрыто:
- Data structures → `programming/12-ALGORITHMS-DATA-STRUCTURES-DEEP-DIVE.md`
- Stack (FIFO/LIFO) → `go/29-GO-STACK-QUEUE-DEEP-DIVE.md`
- Sorting algorithms → `operating-system/08-SORTING-ALGORITHMS-DEEP-DIVE.md`
- Search algorithms → `programming/12-ALGORITHMS-DATA-STRUCTURES-DEEP-DIVE.md`
- OOP benefits → `programming/01-OOP-DEEP-DIVE.md`
- Programming paradigms → `programming/01-OOP-DEEP-DIVE.md`, `programming/02-FUNCTIONAL-PROGRAMMING-DEEP-DIVE.md`
- Big O Notation → `programming/12-ALGORITHMS-DATA-STRUCTURES-DEEP-DIVE.md`
- Process vs Thread → `operating-system/03-PROCESSES-THREADS-DEEP-DIVE.md`

#### ❌ Отсутствует:
- **Work Environment for Software Development** - описание рабочей среды разработки

---

### 2. Intermediate Questions

#### Databases

#### ✅ Уже покрыто:
- SQL databases → `database/` (множество файлов)
- WHERE vs HAVING → частично в `database/14-QUERY-OPTIMIZATION-DEEP-DIVE.md`
- Slow query problem → `database/14-QUERY-OPTIMIZATION-DEEP-DIVE.md`
- SQL vs NoSQL → `database/30-NOSQL-DATABASES-DEEP-DIVE.md`
- NoSQL databases → `database/30-NOSQL-DATABASES-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **WHERE vs HAVING Clauses** - детальное объяснение различий
2. **Semi-Structured Logs Storage** - хранение потоков полуструктурированных логов
3. **Big Data Technologies** - технологии больших данных (Hadoop, Spark, etc.)

#### Design Patterns and Architectural stuff

#### ✅ Уже покрыто:
- Design Patterns → `programming/03-DESIGN-PATTERNS-DEEP-DIVE.md`, `programming/13-DESIGN-PATTERNS-ADVANCED-DEEP-DIVE.md`
- ORM → `go/72-GO-ORM-PATTERNS-DEEP-DIVE.md`
- Pub/Sub pattern → `networking/24-MESSAGE-QUEUES-DEEP-DIVE.md`, `networking/56-EVENT-STREAMING-PUB-SUB-DEEP-DIVE.md`
- REST → `networking/27-REST-API-PRINCIPLES-DEEP-DIVE.md`
- RESTful characteristics → `networking/27-REST-API-PRINCIPLES-DEEP-DIVE.md`
- HTTP verbs → `networking/38-HTTP-METHODS-DEEP-DIVE.md`
- HTTP status codes → `networking/28-HTTP-STATUS-CODES-DEEP-DIVE.md`
- IPC protocols → `operating-system/16-INTER-PROCESS-COMMUNICATION-DEEP-DIVE.md`
- API Gateway → `networking/25-API-GATEWAY-DEEP-DIVE.md`
- Scaling monolithic app → `software-development/05-MONOLITH-MICROSERVICES-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **SOA (Service-Oriented Architecture)** - сервис-ориентированная архитектура
2. **MQ Brokers** - детально про message queue брокеры
3. **ESB (Enterprise Service Bus)** - корпоративная шина сервисов
4. **SSO (Single Sign-On)** - единый вход, протоколы и технологии

---

### 3. Advanced Questions

#### Networking

#### ✅ Уже покрыто:
- TCP vs UDP → `networking/01-TCP-IP-DEEP-DIVE.md`
- OSI model → `networking/33-NETWORK-PROTOCOLS-FUNDAMENTALS-DEEP-DIVE.md`
- CDN → `networking/07-CDN-DEEP-DIVE.md`
- Reverse proxy → частично в `networking/25-API-GATEWAY-DEEP-DIVE.md`
- HTTP/HTTPS ports → частично в `networking/02-HTTP-HTTPS-DEEP-DIVE.md`
- CORS → частично в `security/06-API-SECURITY-BEST-PRACTICES-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Reserved Network Segments** - зарезервированные сетевые сегменты (private IP ranges)
2. **DMZ (Demilitarized Zone)** - демилитаризованная зона
3. **Protocol Ports** - детально про порты протоколов (HTTP, HTTPS, SSH, DNS, FTP)
4. **DHCP Protocol** - как работает DHCP
5. **CORS Deep Dive** - детальное объяснение CORS

#### Cloud Computing

#### ✅ Уже покрыто:
- Virtualization vs Containerization → `operating-system/32-OPERATING-SYSTEM-VIRTUALIZATION-DEEP-DIVE.md`, `software-development/20-CONTAINERIZATION-DOCKER-KUBERNETES-DEEP-DIVE.md`
- Sharding → `database/15-DATABASE-SHARDING-DEEP-DIVE.md`
- HA (High Availability) → частично в различных файлах
- Elasticity → частично в `software-development/29-SOFTWARE-SCALABILITY-DEEP-DIVE.md`
- Network latency → частично в `networking/01-TCP-IP-DEEP-DIVE.md`
- Serverless → частично в `software-development/16-DISTRIBUTED-SYSTEMS-PATTERNS-DEEP-DIVE.md`
- Auto scaling → частично в `software-development/29-SOFTWARE-SCALABILITY-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **IaaS, PaaS, SaaS, FaaS** - детальное объяснение моделей облачных сервисов
2. **Object Storage** - объектное хранилище (S3, Azure Blob, etc.)
3. **12 Factor App** - принципы 12-factor приложения
4. **Cloud Orchestration** - облачная оркестрация (Kubernetes, etc.)

#### Security

#### ✅ Уже покрыто:
- SQL injection → `security/06-API-SECURITY-BEST-PRACTICES-DEEP-DIVE.md`, `security/07-SECURITY-VULNERABILITIES-DEEP-DIVE.md`
- DoS/DDoS → `security/04-SECURITY.md` (overview)
- Firewall → `security/08-NETWORK-SECURITY-DEEP-DIVE.md`
- XSS → `security/06-API-SECURITY-BEST-PRACTICES-DEEP-DIVE.md`, `security/07-SECURITY-VULNERABILITIES-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **Phishing Attacks** - фишинговые атаки
2. **XSS Deep Dive** - детальное объяснение XSS

---

### 4. Soft Skills

#### ❌ Не добавляем - это не технические вопросы

---

### 5. Frontend

#### ❌ Не добавляем - это frontend вопросы, не backend

---

### 6. DevOps

#### ✅ Уже покрыто:
- Versioning → `software-development/32-SOFTWARE-VERSIONING-DEEP-DIVE.md`
- Development workflow → частично в `software-development/10-GIT-BRANCHING-STRATEGIES-DEEP-DIVE.md`
- CI/CD → `software-development/04-CICD-DEEP-DIVE.md`
- Continuous Deployment → `software-development/04-CICD-DEEP-DIVE.md`
- Monitoring tools → `operating-system/11-OBSERVABILITY-DEEP-DIVE.md`
- Orchestration → `software-development/20-CONTAINERIZATION-DOCKER-KUBERNETES-DEEP-DIVE.md`
- VM vs Container → `operating-system/32-OPERATING-SYSTEM-VIRTUALIZATION-DEEP-DIVE.md`

#### ❌ Отсутствует:
1. **IaC (Infrastructure as Code)** - инфраструктура как код (Terraform, CloudFormation, etc.)
2. **Build Automation Tools** - инструменты автоматизации сборки (Maven, Gradle, Make, etc.)

---

### 7. Best Practices

#### ❌ Отсутствует:
1. **Secure Environment Variables Storage** - безопасное хранение переменных окружения

---

### 8. Core Tech Questions

#### ✅ Уже покрыто (концепции):
- Docker → `software-development/20-CONTAINERIZATION-DOCKER-KUBERNETES-DEEP-DIVE.md`
- Kubernetes → `software-development/20-CONTAINERIZATION-DOCKER-KUBERNETES-DEEP-DIVE.md`
- MongoDB → `database/30-NOSQL-DATABASES-DEEP-DIVE.md`

#### ❌ Отсутствует (детальные темы):
1. **Ansible Deep Dive** - детально про Ansible
2. **Elasticsearch Deep Dive** - детально про Elasticsearch
3. **AWS Deep Dive** - детально про AWS сервисы
4. **Azure Deep Dive** - детально про Azure сервисы
5. **Google Cloud Deep Dive** - детально про GCP сервисы
6. **OpenStack Deep Dive** - детально про OpenStack
7. **OpenShift Deep Dive** - детально про OpenShift

**Примечание**: Эти темы очень обширные. Можно добавить обзорные темы или пропустить, так как это больше про конкретные технологии, а не про общие концепции backend.

---

## Итоговый список недостающих тем

### Высокий приоритет (важные для backend):
1. **WHERE vs HAVING SQL Clauses** - детальное объяснение
2. **Semi-Structured Logs Storage** - хранение потоков логов
3. **SOA (Service-Oriented Architecture)** - сервис-ориентированная архитектура
4. **SSO (Single Sign-On)** - единый вход
5. **DMZ (Demilitarized Zone)** - демилитаризованная зона
6. **DHCP Protocol** - протокол DHCP
7. **CORS Deep Dive** - детальное объяснение CORS
8. **IaaS, PaaS, SaaS, FaaS** - модели облачных сервисов
9. **Object Storage** - объектное хранилище
10. **12 Factor App** - принципы 12-factor
11. **IaC (Infrastructure as Code)** - инфраструктура как код
12. **Secure Environment Variables Storage** - безопасное хранение переменных окружения

### Средний приоритет:
13. **Big Data Technologies** - технологии больших данных
14. **MQ Brokers Deep Dive** - детально про message queue брокеры
15. **ESB (Enterprise Service Bus)** - корпоративная шина сервисов
16. **Reserved Network Segments** - зарезервированные сетевые сегменты
17. **Protocol Ports** - порты протоколов
18. **Cloud Orchestration** - облачная оркестрация
19. **Phishing Attacks** - фишинговые атаки
20. **XSS Deep Dive** - детальное объяснение XSS
21. **Build Automation Tools** - инструменты автоматизации сборки

### Низкий приоритет (специфичные технологии):
22. **Ansible Deep Dive** - если нужен Ansible
23. **Elasticsearch Deep Dive** - если нужен Elasticsearch
24. **AWS/Azure/GCP Deep Dive** - если нужны облачные платформы
25. **OpenStack/OpenShift Deep Dive** - если нужны эти технологии

---

## Рекомендации

### Добавить обязательно (высокий приоритет):
1. WHERE vs HAVING SQL Clauses Deep Dive
2. Semi-Structured Logs Storage Deep Dive
3. SOA (Service-Oriented Architecture) Deep Dive
4. SSO (Single Sign-On) Deep Dive
5. DMZ (Demilitarized Zone) Deep Dive
6. DHCP Protocol Deep Dive
7. CORS Deep Dive
8. IaaS, PaaS, SaaS, FaaS Deep Dive
9. Object Storage Deep Dive
10. 12 Factor App Deep Dive
11. IaC (Infrastructure as Code) Deep Dive
12. Secure Environment Variables Storage Deep Dive

### Добавить по желанию (средний приоритет):
13. Big Data Technologies Deep Dive
14. MQ Brokers Deep Dive (детально)
15. ESB (Enterprise Service Bus) Deep Dive
16. Reserved Network Segments Deep Dive
17. Protocol Ports Deep Dive
18. Cloud Orchestration Deep Dive
19. Phishing Attacks Deep Dive
20. XSS Deep Dive (детально)
21. Build Automation Tools Deep Dive

### Можно пропустить:
- Специфичные технологии (Ansible, Elasticsearch, AWS/Azure/GCP детально)
- Frontend вопросы
- Soft Skills вопросы

