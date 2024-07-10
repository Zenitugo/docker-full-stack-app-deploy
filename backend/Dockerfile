FROM python:3.11-slim

# Install Poetry
RUN pip install poetry

# Set the working directory
WORKDIR /application

# Copy pyproject.toml and poetry.lock to the working directory
COPY pyproject.toml poetry.lock /application

# Install dependencies using Poetry
RUN poetry config virtualenvs.create false && \
    poetry install --no-root


# Path variable for poetry
ENV PATH="/root/.local/bin:$PATH"

# Copy the rest of the application cosudode into the container
COPY . .

# Set the environmental path
ENV PYTHONPATH=/application

# Change the permissions
RUN chmod +x /application/prestart.sh

# Expose port
EXPOSE 8080

#Run the prestart script and start the application
CMD ["sh", "-c", "./prestart.sh && poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000"]
