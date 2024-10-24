# Lab 4 : Dockerizing a Python Calculator Application with a React Web Interface



**_ECSE 437- Software Delivery_**\
**_Due:_** **_Nov. 07th, 2024_**

## Objectives

- Learn how to Dockerize a Python calculator application with a React front-end.
- Deploy the application to a cloud platform (e.g., Azure).

## Create the Python Backend Calculator API

### Step 1: Set Up the Backend Folder
First, create a new directory for your project called `backend`. Inside this folder, create a file named `app.py` with the following content:

```python
from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/calculate', methods=['POST'])
def calculate():
    data = request.get_json()
    num1 = data.get('num1')
    num2 = data.get('num2')
    operation = data.get('operation')

    if not all([num1, num2, operation]):
        return jsonify({'error': 'Missing data'}), 400

    try:
        num1 = float(num1)
        num2 = float(num2)
    except ValueError:
        return jsonify({'error': 'Invalid input. Please enter numeric values.'}), 400

    if operation == 'add':
        result = num1 + num2
    elif operation == 'subtract':
        result = num1 - num2
    elif operation == 'multiply':
        result = num1 * num2
    elif operation == 'divide':
        if num2 == 0:
            return jsonify({'error': 'Division by zero'}), 400
        result = num1 / num2
    else:
        return jsonify({'error': 'Invalid operation'}), 400

    return jsonify({'result': result})

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

This script defines a python backend that performs addition, subtraction, multiplication, and division based on incoming POST requests.

## Set Up Docker for the Python API

### Step 2: Create the Dockerfile for the Backend
Next, you'll need to create a `Dockerfile` to containerize the Python API. In the `backend` directory, create a file named `Dockerfile` and add the following content:

```dockerfile
# Step 1: Use an official Python runtime as a base image
FROM python:3.10-slim

# Step 2: Set the working directory in the container
WORKDIR /app

# Step 3: Copy the current directory contents into the container at /app
COPY . /app

# Step 4: Install necessary Python packages
RUN pip install --no-cache-dir flask flask-cors

# Step 5: Expose the port that the Flask application will run on
EXPOSE 5000

# Step 6: Run the Flask application
CMD ["python", "app.py"]
```

### Explanation of Each Step:
1. **Base Image**:
   - `FROM python:3.10-slim`: This line tells Docker to use the official Python image version 3.10-slim as the base image. Slim images are lighter and help reduce the overall container size.

2. **Set Working Directory**:
   - `WORKDIR /app`: This sets the working directory inside the container to `/app`. This is where all subsequent operations will take place.

3. **Copy Application Files**:
   - `COPY . /app`: Copies the current directory (`backend`) contents into `/app` in the container.

4. **Install Dependencies**:
   - `RUN pip install --no-cache-dir flask flask-cors`: Installs the necessary Python packages (`flask` and `flask-cors`). The `--no-cache-dir` flag prevents caching, keeping the image small.

5. **Expose Application Port**:
   - `EXPOSE 5000`: This tells Docker that the application inside the container will use port `5000`. This helps with port mapping when running the container.

6. **Run the Application**:
   - `CMD ["python", "app.py"]`: This is the command that will be run when the container starts, launching the Flask application.

## Create a .dockerignore File

### Step 3: Create a `.dockerignore` File
Create a `.dockerignore` file in the `backend` folder with the following content:

```
__pycache__
*.pyc
*.pyo
*.pyd
.env
```

### Explanation:
- **Ignore Files**: The `.dockerignore` file helps reduce the size of the Docker image by excluding unnecessary files, such as Python bytecode (`*.pyc`, `__pycache__`), and environment files (`.env`). This keeps the image lighter and more secure.

## Build the Docker Image for the Backend

### Step 4: Build the Docker Image
Open your terminal, navigate to the `backend` directory, and run the following command to build the Docker image:

```sh
docker build -t calculator-frontend .
```

### Explanation:
- `docker build`: This command is used to build the Docker image.
- `-t calculator-frontend`: The `-t` flag tags the image with the name `calculator-frontend`.
- `.`: This indicates that the Dockerfile is in the current directory.

## Run the Docker Container for the Backend

### Step 5: Run the Container
To test that the backend works correctly, run the container using the following command:

```sh
docker run -p 5000:5000 calculator-frontend
```

### Explanation:
- `docker run`: This command runs the Docker container.
- `-p 5000:5000`: This maps port `5000` on your machine to port `5000` in the container, allowing you to access the calculator backend.
- `calculator-frontend`: The name of the Docker image to run.

## Test the Calculator API

### Step 6: Test Using PowerShell
To test the Calculator API, use the following PowerShell command:

```powershell
Invoke-WebRequest -Uri http://localhost:5000/calculate -Method POST -Headers @{ "Content-Type" = "application/json" } -Body '{"num1":10,"num2":5,"operation":"add"}'
```

This sends a POST request to the `/calculate` endpoint and should return a result of `15` if the setup is correct.

## Create the React Front-End

### Step 7: Set Up the Frontend Folder
Create the React app by running (make sure you have Node.js installed for this step):

```sh
npx create-react-app frontend
```

Navigate to the `frontend` directory and update the `src/App.js` file with the following content:

```javascript
import React, { useState } from 'react';
import axios from 'axios';
import './App.css';

function App() {
  const [num1, setNum1] = useState('');
  const [num2, setNum2] = useState('');
  const [operation, setOperation] = useState('add');
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);

  const calculate = async () => {
    try {
      const response = await axios.post('https://<your-api-url>/calculate', {
        num1,
        num2,
        operation
      });
      setResult(response.data.result);
      setError(null);
    } catch (err) {
      setError(err.response ? err.response.data.error : 'Error calculating result');
      setResult(null);
    }
  };

  return (
    <div className="App">
      <h1>Calculator</h1>
      <input type="number" value={num1} onChange={(e) => setNum1(e.target.value)} placeholder="Enter first number" />
      <select value={operation} onChange={(e) => setOperation(e.target.value)}>
        <option value="add">+</option>
        <option value="subtract">-</option>
        <option value="multiply">*</option>
        <option value="divide">/</option>
      </select>
      <input type="number" value={num2} onChange={(e) => setNum2(e.target.value)} placeholder="Enter second number" />
      <button onClick={calculate}>Calculate</button>
      {result !== null && <h2>Result: {result}</h2>}
      {error && <h2 style={{ color: 'red' }}>{error}</h2>}
    </div>
  );
}

export default App;
```

## Dockerize the React Front-End

### Step 8: Create a Dockerfile for the Frontend
Create a Dockerfile inside the `frontend` directory with the following content:

```dockerfile
# Use the official Node.js image as the base image for building the app
FROM node:16 as build

# Set the working directory in the container
WORKDIR /app

# Copy the package.json and package-lock.json files into the container
COPY package*.json ./

# Install the dependencies
RUN npm install

# Copy the rest of the application source code
COPY . .

# Build the application for production
RUN npm run build

# Use the official Nginx image to serve the built app
FROM nginx:alpine

# Copy the build output to the Nginx HTML directory
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80 to the world outside this container
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Explanation:
1. **Base Image for Building**:
   - `FROM node:16 as build`: Use the official Node.js image version 16 for building the React app.

2. **Working Directory**:
   - `WORKDIR /app`: Sets the working directory in the container.

3. **Copy Files and Install Dependencies**:
   - `COPY package*.json ./`: Copies `package.json` and `package-lock.json` for installing dependencies.
   - `RUN npm install`: Installs all dependencies for the React project.

4. **Copy Source Code and Build**:
   - `COPY . .`: Copies the entire React app into the container.
   - `RUN npm run build`: Builds the React app for production.

5. **Serve with Nginx**:
   - `FROM nginx:alpine`: Uses the Nginx image to serve the built app.
   - `COPY --from=build /app/build /usr/share/nginx/html`: Copies the build output from the first stage to the Nginx directory.
   - `EXPOSE 80`: Exposes port `80`.
   - `CMD ["nginx", "-g", "daemon off;"]`: Runs Nginx.

## Build and Test the Docker Image for the React Front-End

### Step 9: Build the Docker Image
Navigate to the `frontend` directory in your terminal and build the Docker image:

```sh
docker build -t react-calculator-frontend .
```

### Step 10: Run and Test the Docker Image
You can now test the newly built image by running it:

```sh
docker run -p 80:80 react-calculator-frontend
```

Visit `localhost:80` through your browser to test out your application.

Note: Don't forget to update the `axios.post` url within your code to correctly point to the backend or the calculator won't work. 

## Deploy to Azure Web App

### Step 11: Push Docker images to Azure Container Registry

- **Create a Student Trial Account**:
  - Go to [Azure for Students](https://azure.microsoft.com/free/students/) and sign up for a student trial account. This provides you with free credits to test and deploy your applications.

- **Deploy Calculator backend**:
  - Go to [Azure Portal](https://portal.azure.com/#home).
  
  ![alt text](/images/add.png)

  - Then use the plus sign to create a resource and search for: Azure Container Registry
  
  ![alt text](/images/container-registry.png)

  - Pick a name for the registry name, use the default values for the rest and create the container registry.
  
  ![alt text](/images/create-registry.png)

  - Then click on “go to resource” and then use “Access Keys” to login to the registry:
  
  ![alt text](/images/registry-overview.png)

  - To login to the registry, you can use the script below:
    
    ```sh
    docker login <your-registry-name>.azurecr.io -u <user> -p <password>
    ```
    ![alt text](/images/registry-access-keys.png)
  
  - Push your Docker image to the registry using the following commands:

    ```sh
    docker tag calculator-api <your-registry-name>.azurecr.io/calculator-api:v1
    docker push <your-registry-name>.azurecr.io/calculator-api:v1
    ```

### Step 12: Deploy the App

To deploy our app, we need to create a Web App using the production dockerfile.

- **Create an Azure Web App**:

  - First go to this address: [Azure Portal](https://portal.azure.com/#home)
  
  ![alt text](/images/add.png)
  - Then use the plus sign to create a resource and search for: Web App
  
  ![alt text](/images/web-app.png)
  - Use the option “Web App for Containers” and create one.
  
  ![alt text](/images/create-webapp-1.png)

  - Set the **PORT** environment variable to `5000` in the Azure Web App configuration. This step is crucial to ensure that Azure routes traffic correctly to your backend.
  
  ![alt text](/images/web-app-environment-variables.png)


## Verify Deployment

- **Test the Backend**:
  - Go to the Azure Portal, find the Web App for the calculator backend, and note down the URL.
  - Test the calculator backend by sending a POST request link as we did earlier in the tutorial.



## Front-End Deployment Challenge

**Challenge**: Now that you have successfully Dockerized and deployed the backend, it's time to deploy the React front-end. Your task is to create an Azure Web App for the front-end container and ensure it communicates with the backend successfully.