# Workshop: Deta Cloud

## Deta Cloud Intro

[Deta](https://deta.sh) ist ein Startup aus Berlin, welches Cloudservices anbietet, die es Entwicklern möglichst einfach machen sollen, ihre Projekte zu realisieren.

## Account

## Deta Commandline Tool

## Deta Micros

## Deta Base

## Workshop Beispiel - Backend

1. Anlegen eines Repositories auf GitHub mit einer README.md
1. In VS Code öffnen
    1. Installieren der devcontainer files für Python 3.9
    1. Dockerfile um Deta Tools ergänzen
    ```Dockerfile
    USER vscode

    RUN curl -fsSL https://get.deta.dev/cli.sh | sh
    RUN echo "export PATH=/home/vscode/.deta/bin:${PATH}" >> /home/vscode/.bashrc
    ```
1. Im Devcontainer öffnen
1. Anlegen eines .gitignore
    ```bash
    wget -O .gitignore https://github.com/github/gitignore/raw/master/Python.gitignore
    ```
1. Anlegen einer Datei requirements.txt und deta.json
    ```
    flask
    flask-cors
    deta
    python-dotenv
    ```
1. Anlegen eines .env
    ```bash
    DETA_KEY=<your key>
    ```
1. Anlegen eines Juyter Notebooks zum demonstrieren
    ```python
    from deta import Deta
    from dotenv import load_dotenv
    import os
    load_dotenv()

    DETA_KEY = os.environ.get('DETA_KEY', None)

    if DETA_KEY:
        deta = Deta(DETA_KEY)
    else:
        deta = Deta()
    ```
    ```python
    users = deta.Base('users')
    users.insert({
        "first_name": "Andre",
        "last_name": "Bell"
    })
    ```
1. Zeigen, dass eine Datenbank angelegt wurde und die Daten eingetragen worden sind.
1. Anlegen eines ```main.py```
    ```python
    # -*- coding: utf-8 -*-

    from deta import Deta
    from dotenv import load_dotenv
    from flask import Flask, request, jsonify
    from flask_cors import CORS
    import os

    load_dotenv()
    DETA_KEY = os.environ.get('DETA_KEY', None)

    if DETA_KEY:
        deta = Deta(DETA_KEY)
    else:
        deta = Deta()

    db_users = deta.Base('users')
    app = Flask(__name__)
    CORS(app)

    # {
    #     "name": str,
    #     "age": int,
    #     "hometown": str
    # }

    @app.route('/users', methods=['GET'])
    def get_users():
        userlist = db_users.fetch()
        return userlist.items

    @app.route('/users', methods=["POST"])
    def create_user():
        name = request.json.get("name")
        age = request.json.get("age")
        hometown = request.json.get("hometown")

        user = db_users.put({
            "name": name,
            "age": age,
            "hometown": hometown
        })
        return jsonify(user, 201)

    @app.route("/users/<key>")
    def get_user(key):
        user = db_users.get(key)
        return user if user else jsonify({"error": "Not found"}, 404)

    @app.route("/users/<key>", methods=["PUT"])
    def update_user():
        user = db_users.put(request.json)
        return user

    @app.route("/users/<key>", methods=["DELETE"])
    def delete_user(key):
        db_users.delete(key)
        return
    ```
1. Demonstration der API im Web oder mit [Hoppscotch](https://hoppscotch.io)
    ```bash
    $ deta new --name homecontrol
    $ deta auth disable
    $ deta deploy
    ```
1. Quellcode anpassen
    1. Datenbank von users auf "lights" wechseln
    1. Endpoint zum lesen des Zustands erzeugen
    ```python
    ```

## 
