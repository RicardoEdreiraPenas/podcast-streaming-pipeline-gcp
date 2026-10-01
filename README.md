# 🎧 Pipeline de streaming en GCP · Recomendaciones de pódcast en tiempo real

![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![Apache Beam](https://img.shields.io/badge/Apache_Beam-Dataflow-F26B21?logo=apache&logoColor=white)
![Pub/Sub](https://img.shields.io/badge/Pub%2FSub-4285F4?logo=googlecloud&logoColor=white)
![BigQuery](https://img.shields.io/badge/BigQuery-669DF6?logo=googlebigquery&logoColor=white)
![Firestore](https://img.shields.io/badge/Firestore-FFCA28?logo=firebase&logoColor=black)
![Cloud Build](https://img.shields.io/badge/Cloud_Build-CI%2FCD-4285F4?logo=googlecloud&logoColor=white)

Pipeline de **Dataflow (Apache Beam)** que procesa en tiempo real los eventos de una plataforma de pódcast: reproducciones, interacción y calidad de audio. Calcula métricas por usuario y por episodio, las guarda en **BigQuery** y genera notificaciones personalizadas que se escriben en **Firestore** y se publican en **Pub/Sub**.

Ejercicio del Máster en Big Data y Cloud de EDEM (2025–2026).

---

## Arquitectura

```mermaid
flowchart LR
    P1[[Pub/Sub<br/>playback]] --> N
    P2[[Pub/Sub<br/>engagement]] --> N
    P3[[Pub/Sub<br/>quality]] --> N
    N[Normalización<br/>y unión de eventos] --> U[Ventanas de sesión<br/>por usuario · 30 s]
    N --> C[Ventanas deslizantes<br/>por episodio · 60 s cada 10 s]
    U -->|métricas| BQ1[(BigQuery<br/>usuarios)]
    U -->|CONTINUE_LISTENING| FS[(Firestore<br/>notificaciones)]
    U -->|CONTINUE_LISTENING| OUT[[Pub/Sub<br/>delivery-events]]
    C -->|métricas| BQ2[(BigQuery<br/>episodios)]
    C -->|TRENDING_NOW| OUT
```

## Qué hace

1. **Ingesta** de tres flujos de eventos desde Pub/Sub y **normalización** a un formato común.
2. **Métricas por usuario** con ventanas de sesión: reproducciones, escuchas completas y una puntuación de interacción.
3. **Métricas por episodio** con ventanas deslizantes, para detectar contenido que sube rápido.
4. **Notificaciones**, separadas de las métricas mediante salidas etiquetadas de Beam:
   - `CONTINUE_LISTENING`: el usuario paró un episodio sin terminarlo; se guarda el minuto exacto para retomarlo.
   - `TRENDING_NOW`: un episodio supera el umbral de reproducciones en la ventana.
5. **Persistencia**: métricas en BigQuery, notificaciones de usuario en Firestore (`users/{user_id}/notifications`) y todas las notificaciones en Pub/Sub para su envío.

## Despliegue

El despliegue está automatizado con **Cloud Build** (`streaming_build.yml`): construye una **Dataflow Flex Template**, sube la imagen a Artifact Registry y lanza el job en streaming.

```bash
gcloud builds submit --config streaming_build.yml .
```

Los nombres de suscripciones, tablas, bucket y región se configuran como *substitutions* en `streaming_build.yml`.

## Estructura

```
├── edem_realtime_recommendation_engine.py   Pipeline de Apache Beam
├── streaming_build.yml                      Build y despliegue con Cloud Build
└── requirements.txt
```

## Autor

**Ricardo Edreira Penas** · Data Analyst · Data Engineer Junior
[LinkedIn](https://www.linkedin.com/in/ricardoedreira) · [GitHub](https://github.com/RicardoEdreiraPenas)
