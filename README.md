# Geo Service-Area Router

A Django REST and PostGIS API for managing service providers, storing polygonal coverage areas, and finding the areas that contain a supplied coordinate.

## Spatial workflow

```text
provider -> one or more polygon service areas
                         ^
                         |
              longitude / latitude query
                         |
                         v
              PostGIS point-in-polygon
```

The lookup creates a GIS point in `(longitude, latitude)` order and filters non-null polygons with PostGIS `contains`.

## Endpoints

- `/testapp/provider/` — provider CRUD.
- `/testapp/service_area/` — service-area CRUD.
- `/testapp/search_service_area/?lat=40.7128&lng=-74.0060` — containing areas.

Missing or non-numeric coordinates return `400`. Successful searches return matching serialized service areas and support the configured REST Framework pagination path.

## Local setup

PostgreSQL with PostGIS and the native geospatial libraries required by GeoDjango must be available.

```bash
cp .env.example .env
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

Update `.env` with local database credentials before migrating.

## Data model

- `Provider` stores contact, language, and currency information.
- `ServiceArea` stores a name, price, optional polygon, and optional provider relation.
- Deleting a provider cascades to its related service areas.

## Verification and design boundaries

- `testapp/tests.py` does not contain behavioral coverage.
- Polygon coordinates and search points rely on the default spatial reference; explicit SRID handling is not documented in the API.
- Provider phone and email fields are unique, while a service area may currently exist without a provider or polygon.
- Authentication, authorization, and production deployment configuration are outside this prototype.
- Django, PostgreSQL, and PostGIS were not run during the portfolio review.

## Next improvements

1. Add request tests for CRUD, coordinate validation, polygon boundaries, and pagination.
2. Document or enforce the spatial reference system at serialization boundaries.
3. Decide whether provider and polygon relations should be mandatory.
4. Add authentication before exposing provider mutations beyond a trusted environment.
