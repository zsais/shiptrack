# Shiptrack

Shiptrack is a Flask-based Python application designed to manage package information. It provides a RESTful API for creating, retrieving, updating, and deleting package details, which are stored in a Cloud SQL database.

## Features

- **Discovery Endpoint**: Provides metadata about the service, including name, version, owners, team, and organization.
- **Liveness & Readiness Probes**: Standard Kubernetes-style endpoints to check the health and readiness of the application.
- **Package Management**: 
    - Create new package entries with dimensions (height, width, depth), weight, and special handling instructions.
    - Retrieve package details by product ID.
    - Update existing package information by package ID.
    - Delete packages by package ID.
- **Database Integration**: Uses SQLAlchemy to interact with a Cloud SQL (PostgreSQL) database.

## Technologies Used

- **Backend**: Python, Flask
- **Database**: Google Cloud SQL (presumably PostgreSQL, given `pg8000` and `psycopg2-binary`)
- **ORM**: SQLAlchemy
- **HTTP Client**: Requests (for internal `discovery` call)
- **WSGI Server**: Gunicorn (implied by `requirements.txt`)
- **Dependencies** (from `requirements.txt`):
    - `cloud-sql-python-connector==1.2.4`
    - `sqlalchemy==1.4.36`
    - `pg8000==1.22.0`
    - `Flask==3.0.0`
    - `gunicorn==20.1.0`
    - `psycopg2-binary`
    - `requests==2.28.1`
    - `CORS`
    - `flask_cors`

## Project Structure

```
shiptrack/
├── .gemini/           # Gemini configuration (inferred)
├── .gitignore         # Specifies intentionally untracked files that Git should ignore
├── README.md          # This file
├── app.py             # Main Flask application logic, API endpoints
├── connect_connector.py # Handles database connection setup using Cloud SQL Python Connector
├── data_model.py      # Defines the SQLAlchemy data model (Package table)
├── packages.json      # Purpose unclear from content, might be sample data or package list
├── requirements.txt   # Python package dependencies
├── tmp/               # Temporary files directory
└── vars.sh            # Shell script for setting environment variables (likely for DB connection)
```

## Setup and Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/zsais/shiptrack.git
    cd shiptrack
    ```

2.  **Set up a Python virtual environment (recommended):**
    ```bash
    python3 -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**
    The application connects to a Google Cloud SQL database. You'll need to set up a Cloud SQL instance (PostgreSQL) and configure the necessary environment variables. The `vars.sh` script likely contains placeholders for these. Common variables needed for `cloud-sql-python-connector` include:
    - `DB_USER`: Database user
    - `DB_PASS`: Database password
    - `DB_NAME`: Database name
    - `INSTANCE_CONNECTION_NAME`: Cloud SQL instance connection name (e.g., `project:region:instance`)

    Source the `vars.sh` script after populating it with your actual credentials (or set them directly in your environment):
    ```bash
    source vars.sh
    ```

5.  **Initialize the database:**
    The `data_model.py` defines the `Package` table. You may need to run a script or use an ORM tool (like Alembic, though not listed in requirements) to create the table schema in your database if it doesn't exist. Alternatively, the application might create it on first run if SQLAlchemy is configured to do so (not evident from the provided code).

## Running the Application

Once the setup is complete, you can run the Flask development server:

```bash
python app.py
```

This will typically start the server on `http://0.0.0.0:8000`.

For production, you would use Gunicorn:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
```

## API Endpoints

All endpoints are relative to the base URL (e.g., `http://localhost:8000`).

### Health & Discovery

-   **GET `/discovery`**: Get service metadata.
    -   **Success Response (200 OK):**
        ```json
        {
            "name": "shipping",
            "version": "1.0",
            "owners": ["ameerabb", "lonestar"],
            "team": "genAIs",
            "organization": "acme"
        }
        ```

-   **GET `/liveness`**: Check if the application is live.
    -   **Success Response (200 OK):**
        ```json
        {"status": "live", "code": 200, "timestamp": 1678886400.000}
        ```

-   **GET `/readiness`**: Check if the application is ready to serve requests.
    -   **Success Response (200 OK):**
        ```json
        {"status": "ready", "code": 200, "timestamp": 1678886400.000}
        ```

### Package Management

-   **POST `/packages`**: Create a new package.
    -   **Request Body:**
        ```json
        {
            "product_id": 123,
            "height": 10.5,
            "width": 5.2,
            "depth": 2.0,
            "weight": 1.5,
            "special_handling_instructions": "Fragile"
        }
        ```
    -   **Success Response (201 Created):**
        ```json
        {"package_id": 1}
        ```
    -   **Error Responses:** `400 Bad Request` for missing or invalid data.

-   **GET `/packages/<product_id>`**: Retrieve package details by product ID.
    -   **Example:** `GET /packages/123`
    -   **Success Response (200 OK):**
        ```json
        {
            "height": 10.5,
            "width": 5.2,
            "depth": 2.0,
            "weight": 1.5,
            "special_handling_instructions": "Fragile"
        }
        ```
    -   **Error Response:** `404 Not Found` if the product ID does not exist.

-   **PUT `/packages/<package_id>`**: Update an existing package by its internal package ID.
    -   **Example:** `PUT /packages/1`
    -   **Request Body (include fields to update):**
        ```json
        {
            "weight": 1.8,
            "special_handling_instructions": "Fragile, This side up"
        }
        ```
    -   **Success Response (200 OK):**
        ```json
        {
            "height": 10.5, /* Unchanged if not provided */
            "width": 5.2,   /* Unchanged if not provided */
            "depth": 2.0,   /* Unchanged if not provided */
            "weight": 1.8,
            "special_handling_instructions": "Fragile, This side up"
        }
        ```
    -   **Error Responses:** `400 Bad Request` for missing or invalid data, `404 Not Found` if package ID does not exist.

-   **DELETE `/packages/<package_id>`**: Delete a package by its internal package ID.
    -   **Example:** `DELETE /packages/1`
    -   **Success Response:** `204 No Content`
    -   **Error Response:** `404 Not Found` if package ID does not exist.

## Contributing

(Details on how to contribute to the project, if applicable. E.g., fork the repo, create a feature branch, submit a pull request.)

## License

(Specify the license for the project, e.g., MIT, Apache 2.0. If no license file is present, this section can be omitted or state "Proprietary".)
