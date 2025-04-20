# AgriSurvey

## Overview

AgriSurvey is a lightweight web application for conducting agricultural field surveys. It lets users record farm details, manage crop information, log soil and water samples, and report pests and diseases. Designed for intuitive, efficient data collection, it empowers better decision‑making and resource allocation in the agricultural sector. Enumerators can quickly capture and save data in the field, then sync to a central database when online.

The backend is built in Python using the Django framework with SQLite for storage, while the frontend uses HTML, CSS, and JavaScript to provide a responsive, interactive user experience.

The application is structured into four main modules:

1. **Auth & Route Assignment**: This module handles user authentication and assigns specific fields or farms to each enumerator.
2. **Farm & Crop Details**: This module allows users to record essential information about the farm, including size, owner details, GPS boundaries, and the crops planted.
3. **Soil & Water Sampling**: This module enables users to log soil parameters such as pH, moisture, and nutrient levels, as well as upload water quality photos and readings.
4. **Pest & Disease Reporting**: This module captures pest sightings and disease symptoms, allowing users to upload photos and text descriptions. It also includes a feature to flag hotspots and alert agronomists.

## Features

- User authentication and role-based access control
- Field assignment for enumerators
- Farm details entry (size, owner info, GPS boundaries)
- Crop information management (planted crops and varieties)
- Soil sampling (pH, moisture, nutrient levels)
- Water sampling (quality photos and readings)
- Pest and disease reporting (photos and text)
- Hotspot flagging and agronomist alerts
- Responsive design for mobile and desktop
- SQLite database for data storage

## Project Scope

- Create initial database schema and models
- Integrate Django ORM with postgreSQL
- Implement simple user authentication
- Dashboard with mockup data
- Build basic UI for show:
    - Route assignment
    - Farm details entry
    - Crop information management
    - Soil sampling
    - Water sampling
- Host the application on a serverless platform (e.g., Fly.io, AWS Lambda)

## Data Model

### 1. Auth & Route Assignment

```yaml
User:
  type: object
  properties:
    id: { type: string, format: uuid }
    name: { type: string }
    email: { type: string, format: email }
    role: { type: string, enum: [admin, enumerator] }
  required: [id, name, email, role]

Route:
  type: object
  properties:
    id: { type: string, format: uuid }
    assignedTo: { type: string, format: uuid } # → User.id
    dateAssigned: { type: string, format: date }
    status: { type: string, enum: [pending, in_progress, complete] }
  required: [id, assignedTo, dateAssigned]
```

### 2. Farm & Crop Details

```yaml
Farm:
  type: object
  properties:
    id: { type: string, format: uuid }
    routeId: { type: string, format: uuid } # → Route.id
    ownerName: { type: string }
    sizeHa: { type: number } # hectares
    boundaryGeo: { type: object } # GeoJSON polygon
    createdAt: { type: string, format: date‑time }
  required: [id, routeId, ownerName, sizeHa]

Crop:
  type: object
  properties:
    id: { type: string, format: uuid }
    farmId: { type: string, format: uuid } # → Farm.id
    cropType: { type: string }
    variety: { type: string }
    plantingDate: { type: string, format: date }
    expectedHarvest: { type: string, format: date }
  required: [id, farmId, cropType, plantingDate]
```

### 3. Soil & Water Sampling

```yaml
SoilSample:
  type: object
  properties:
    id: { type: string, format: uuid }
    farmId: { type: string, format: uuid } # → Farm.id
    sampleDate: { type: string, format: date }
    pH: { type: number }
    moisturePct: { type: number }
    nutrients: # N, P, K values
      type: object
      properties:
        N: { type: number }
        P: { type: number }
        K: { type: number }
    photoUrl: { type: string, format: uri }
  required: [id, farmId, sampleDate, pH]

WaterSample:
  type: object
  properties:
    id: { type: string, format: uuid }
    farmId: { type: string, format: uuid } # → Farm.id
    sampleDate: { type: string, format: date }
    pH: { type: number }
    turbidity: { type: number }
    photoUrl: { type: string, format: uri }
  required: [id, farmId, sampleDate, pH]
```

### 4. Pest & Disease Reporting

```yaml
PestDiseaseReport:
  type: object
  properties:
    id: { type: string, format: uuid }
    farmId: { type: string, format: uuid } # → Farm.id
    reportDate: { type: string, format: date }
    category: { type: string, enum: [pest, disease] }
    name: { type: string }
    severity: { type: string, enum: [low, medium, high] }
    description: { type: string }
    photoUrl: { type: string, format: uri }
    location: # GPS optional override
      type: object
      properties:
        lat: { type: number }
        lng: { type: number }
  required: [id, farmId, reportDate, category, name]
```

### ER Diagram

```mermaid
erDiagram
    USER ||--o{ ROUTE            : assigns
    ROUTE ||--o{ FARM            : covers
    FARM  ||--o{ CROP            : grows
    FARM  ||--o{ SOIL_SAMPLE     : has
    FARM  ||--o{ WATER_SAMPLE    : has
    FARM  ||--o{ PEST_DISEASE_REPORT : records
```

## UI Flow

```mermaid
flowchart TD
    %% Authentication
    A[Login Screen]
    A --> B[Dashboard]

    %% Dashboard & Routes
    B --> C[Route List]
    C --> D[Farm List]

    %% Farm Survey Entry (Overview)
    D --> E[Farm Survey Overview]
    E -->|"Farm Info"|F1[Farm Info Form]
    E -->|"Crops"|F2[Crop Details Form]
    E -->|"Samples"|F3[Sampling Menu]
    E -->|"Pests"|F4[Pest/Disease Report]

    %% Sampling submenu
    F3 --> G1[Soil Sample Form]
    F3 --> G2[Water Sample Form]

    %% Back links / Flow loops
    F1 --> E
    F2 --> E
    G1 --> F3
    G2 --> F3
    F4 --> E

    %% Navigation back up
    E --> D
    D --> C
    C --> B

    %% Optional global nav
    B --> H[Sync Status & Settings]
    H --> B

    B --> I[Reports]
    I --> B
    I --> J[Export Data]
    J --> I
    J --> B
```

## UI/UX Design

### 1. Authentication

![Login Screen](./screenshots/login.png)

### 2. Dashboard

![Dashboard](./screenshots/dashboard.png)

### 3. Route List

![Route List](./screenshots/route_list.png)

### 4. Farm List

![Farm List](./screenshots/farm_list.png)

### 5. Sampling

### 5.1 Soil Sample

![Soil Sample](./screenshots/soil_sample.png)

### 5.2 Water Sample

![Water Sample](./screenshots/water_sample.png)

### 6. Pest/Disease Report

![Pest/Disease Report](./screenshots/pest_report.png)
