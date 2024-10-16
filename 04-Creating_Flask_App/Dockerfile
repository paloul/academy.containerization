FROM python:3.9-slim-buster

# Set the working directory
WORKDIR /app

# Copy our necessary files for the application
COPY ./flask_app.py /app
COPY ./requirements.txt /app

# Install our dependencies
RUN pip install -r requirements.txt

# Expose the 5000 port on the container.
EXPOSE 5000

ENV FLASK_APP=flask_app.py

CMD ["flask", "run", "--host", "0.0.0.0"]