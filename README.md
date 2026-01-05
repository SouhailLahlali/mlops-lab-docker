# Docker – Conteneurisation et déploiement de l’API Churn

# Étape 1 : Vérifier l’installation de Docker

```bash
docker --version
docker ps
```

![Docker version](https://github.com/user-attachments/assets/fe807661-712f-4316-82ca-8a47db94331b)
![Docker daemon](https://github.com/user-attachments/assets/7ed69d76-54af-4ed7-8302-cbd6b0bf9bf9)

---

# Étape 2 : Lancer un serveur Nginx dans un conteneur

```bash
docker run -d -p 8080:80 --name demo-nginx nginx
```

Accéder à : [http://localhost:8080](http://localhost:8080)

```bash
docker ps
docker stop demo-nginx
docker rm demo-nginx
```

![Nginx run](https://github.com/user-attachments/assets/66289554-ed38-49e7-8a9b-94fa7ab39578)
![Nginx browser](https://github.com/user-attachments/assets/fc9fabc0-ce20-417c-afb2-574a6b616b63)
![Docker ps](https://github.com/user-attachments/assets/f28c29e2-ac67-435c-aae1-33718f290804)
![Docker stop rm](https://github.com/user-attachments/assets/aff90236-571a-4eb1-824e-9e1973e910c2)

---

# Étape 3 : Ouvrir un shell Linux isolé dans un conteneur

```bash
docker run -it --name demo-ubuntu ubuntu bash
```

Dans le conteneur :

```bash
ls
cat /etc/os-release
pwd
apt-get update
apt-get install -y curl
exit
```

```bash
docker ps -a
docker rm demo-ubuntu
```

![Ubuntu shell](https://github.com/user-attachments/assets/66c12aa2-7045-4697-b64a-3fde6e255b58)
![Ubuntu commands](https://github.com/user-attachments/assets/f4fda86b-d236-475d-9441-20c6e5854b2f)
![Docker ps a](https://github.com/user-attachments/assets/13a531cb-5f2c-4934-9de8-3ce79f4009ad)
![Docker rm ubuntu](https://github.com/user-attachments/assets/53fd0d63-80da-471c-beee-d413266fea70)

---

# Étape 4 : Comprendre la structure d’une commande docker run

```bash
docker run [options] image [commande] [arguments]
```

Exemple :

```bash
docker run -d \
  --name demo-nginx \
  -p 8080:80 \
  nginx
```

* `-d` : mode détaché (arrière-plan)
* `--name` : nom du conteneur
* `-p` : mapping des ports
* `nginx` : image utilisée

```bash
docker stop demo-nginx
docker rm demo-nginx
```

![Docker run structure](https://github.com/user-attachments/assets/0cb1f07b-b3b8-4f44-8dd9-721c09277c71)

---

# Étape 5 : Conteneuriser l’API churn du projet mlops-lab-01

```bash
cd mlops-lab-01
```

Arborescence minimale attendue :

```
mlops-lab-01/
├── data/
├── logs/
├── models/
├── registry/
└── src/
```

Test local (optionnel) :

```bash
python src/api.py
# ou
uvicorn src.api:app --reload
```

![API local](https://github.com/user-attachments/assets/db985957-27ac-427b-b911-d7e44c3ad01c)

---

# Étape 6 : Créer le fichier requirements.txt

```text
fastapi
uvicorn[standard]
pydantic
scikit-learn
pandas
numpy
joblib
```

![Requirements](https://github.com/user-attachments/assets/c5aa3cb3-a904-455d-b645-bcf10e6f4dca)

---

# Étape 7 : Créer le Dockerfile de l’API churn

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "src.api:app", "--host", "0.0.0.0", "--port", "8000"]
```

![Dockerfile](https://github.com/user-attachments/assets/f18414e3-68fe-4563-9061-02a46f6d86e5)

---

# Étape 8 : Préparer un modèle actif

```bash
ls models
cat registry/current_model.txt
```

Si vide :

```bash
python -c "from src.train import main; main(version='v1', gate_f1=0.60)"
```

![Current model](https://github.com/user-attachments/assets/c7e918ac-1e0f-4755-8c9b-3189a0b182e5)

---

# Étape 9 : Construire l’image Docker

```bash
docker build -t churn-api:latest .
docker images
```

![Docker build](https://github.com/user-attachments/assets/d5a48d6b-89f6-4a08-8aa4-41ff156f42f5)
![Docker images](https://github.com/user-attachments/assets/403da9a2-b3d4-499b-8af6-f2836436095f)

---

# Étape 10 : Lancer le conteneur de l’API churn

```bash
docker run -d -p 8000:8000 --name churn-api-demo churn-api:latest
```

---

# Étape 11 : Vérifier les logs à l’intérieur du conteneur

```bash
docker exec -it churn-api-demo ls
```

![Logs conteneur](https://github.com/user-attachments/assets/16c9f551-627f-4472-9375-4889fa2bc6aa)

---

# Étape 12 : Orchestration locale avec Docker Compose

Créer le fichier `docker-compose.yml` :

```yaml
version: "3.9"

services:
  churn-api:
    build: .
    image: churn-api:latest
    container_name: churn-api-compose
    ports:
      - "8000:8000"
    environment:
      LOG_LEVEL: "info"
    volumes:
      - ./logs:/app/logs
```

![Docker compose file](https://github.com/user-attachments/assets/155aa379-d88b-4f1f-abc5-a138d4f11e23)

---

# Étape 13 : Démarrer l’API avec Docker Compose

```bash
docker compose up
```

Tester :

```bash
curl http://localhost:8000/health
```

![Compose up](https://github.com/user-attachments/assets/d9df18e1-b1bd-4cf8-a6a0-a49de7283381)

---

# Étape 14 : Mode détaché et observation des logs

```bash
docker compose up -d
docker ps
docker compose logs -f churn-api
```

Tester `/health` et `/predict` pendant que les logs défilent.

![Compose logs 1](https://github.com/user-attachments/assets/72d2c548-391e-4d7d-aea2-4877016d66d5)
![Compose logs 2](https://github.com/user-attachments/assets/e784c458-9ec0-4ccb-ac5f-0cf36b7d3118)

Arrêter les services :

```bash
docker compose down
```

![Compose down](https://github.com/user-attachments/assets/d4e2cea4-9651-43be-8c72-581c9ebf54db)
