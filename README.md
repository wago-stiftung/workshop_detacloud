# Workshop: Deta Cloud

## Deta Cloud Intro

[Deta](https://deta.sh) ist ein Startup aus Berlin, welches Cloudservices anbietet, die es Entwicklern möglichst einfach machen sollen, ihre Projekte zu realisieren.

## Account

Auf [Deta](https://deta.sh) das dashboard anklicken und falls noch nicht geschehen einen Account erstellen. Falls schon ein Account vorhanden ist unterhalb des Registrierungsformulars auf "Sign in" klicken und anmelden.

## Deta Commandline Tool

Um während der Entwicklung mit der Deta cloud zu arbeiten ist ein Kommandozeilentool erforderlich. Dies kann entweder direkt im Betriebssystem des Entwicklungsrechners installiert werden oder bspw. im Development-Container in VS Code.

Die Installationsschritte sind in der [Doku](https://docs.deta.sh/docs/cli/install) beschrieben.

## Deta Micros

Ein Deta Micro ist soetwas wie eine kleine virtuelle Maschine, auf der die eigene App im Internet läuft. Dies kann bspw. in Python oder in node.js geschrieben werden. In diesem Workshop werden wir eine ReST API in Python mit dem Flask Framework entwickeln.

## Deta Base

Die Deta Base ist eine Datenbank. Aus der eigenen App können leicht mehrere Datenbanken angelegt werden. In diesem Beispiel werden wir den Zustand von Lichtern in einer Datenbank halten und über die ReST API an- und ausschaltbar machen.

## Workshop Beispiel - Backend

1. Anlegen eines Repositories auf GitHub mit einer README.md und mit GitHub Desktop clonen
1. In VS Code öffnen
1. OPTION Docker:
    1. Installieren der devcontainer files für Python 3.9
    1. Dockerfile um Deta Tools ergänzen
        ```Dockerfile
        USER vscode

        RUN curl -fsSL https://get.deta.dev/cli.sh | sh
        RUN echo "export PATH=/home/vscode/.deta/bin:${PATH}" >> /home/vscode/.bashrc
        ```
    1. Im Devcontainer öffnen
1. OPTION Betriebsystem (Windows):
    1. Installation ausführen mit
        ```powershell
        iwr https://get.deta.dev/cli.ps1 -useb | iex
        ```
    1. VS Code Path der Powershell anpassen. File -> Settings
        `terminal.integrated.env` editieren und ergänzen
        ```json
        "terminal.integrated.env.windows": {
            "PATH": "${env:PATH};C:\\users\\u013728\\.deta\\bin"
        }
        ```
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
1. Anlegen einer .env Datei. Der auth key ist sozusagen das "Passwort", um mit der deta API auf die Cloud zugreifen zu können.
    ```bash
    DETA_KEY=<your key>
    ```
1. Anlegen eines Jupyter Notebooks zum demonstrieren
    Diese Zelle lädt für die Entwicklung auf dem eigenen Rechner den auth key über dotenv. Sobald der Code in der Cloudinstanz auf einem Deta micro läuft ist der key nicht mehr erforderlich. Dafür ist der Aufruf `deta = Deta()`. 
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
    Als Beispiel für die Cloud wird hier eine Datenbank (Deta base) mit Namen 'users' erzeugt, in der hier dann ein Eintrag eingefügt wird.
    ```python
    users = deta.Base('users')
    users.insert({
        "first_name": "Andre",
        "last_name": "Bell"
    })
    ```
1. Im Deta Dashboard ist nun eine Datenbank angelegt worden. Hier kann man sich den Inhalt ansehen.
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

    app = Flask(__name__)
    CORS(app)

    @app.route('/', methods=['GET'])
    def get_root():
        return 'WAGO Stiftung - Makeathon 2021 - Workshop Deta Cloud'
    ```
1. Demonstration der API im Web oder mit [Hoppscotch](https://hoppscotch.io)
    Zuerst erzeugen wir einen Deta micro mit `deta new`. Durch `deta auth` wird der Endpunkt im Netz für jeden ohne Authentifizierung erreichbar. Lokale Änderungen am Code können mit `deta deploy` immer wieder in die Cloud veröffentlicht werden.
    ```bash
    $ deta new --name homecontrol
    $ deta auth disable
    $ deta deploy
    ```
1. Ergänzen einer Datenbank `lights` und eines Lesezugriffs im Endpoint
    ```python
    #########################################################
    ## LIGHTS - READ
    #########################################################

    LIGHTNAMES = [
        'bad1',
        'bad2',
        'bad3',
        'couchtisch',
        'esstisch',
        'flur',
        'gaestezimmer',
        'garderobe',
        'kueche',
        'schlafzimmer',
        ]

    db_lights = deta.Base('lights')
    db_lights.put_many([dict(key=n, value=False) for n in LIGHTNAMES])
    
    @app.route('/lights', methods=['GET'])
    def get_lights():
        lightlist = db_lights.fetch()
        result = list(
            map(
                lambda d: (d['key'], d['value']),
                lightlist.items
            )
        )
        return jsonify(dict(result))

    @app.route('/lights/<name>', methods=['GET'])
    def get_light(name):
        light = db_lights.get(name)
        return jsonify(light['value']) if light else jsonify({"error": "Not found"}, 404)
    ```
    Nach einem `deta deploy` können mit einem Browserfenster auf die Datenbank und einem Browserfenster auf den Endpoint die Abfragen verglichen werden.
1. Als letzten Punkt im Deta micro soll eine Möglichkeit eingebaut werden ein Licht an- oder auszuschalten. Das erfolgt dann bspw. mit Hoppscotch.io zur Demonstration.
    ```python
    #########################################################
    ## LIGHTS - WRITE
    #########################################################

    @app.route('/lights/<name>', methods=['POST'])
    def set_light(name):
        value = request.json
        if type(value) == bool:
            if db_lights.get(name):
                db_lights.put(value, name)
                return jsonify(value, 201)
        return jsonify({"error": "Not found"}, 404)
    ```
    Das ganze wird wieder mit `deta deploy` online gestellt.

## Workshop Beispiel - Frontend

Im Repo muss der korrekte Endpoint in die index.html eingetragen werden. Danach kann die erzeugte Datei (GitHub Pages) geladen und demonstriert werden. Ggf. parallel Dashboard und Frontend...
