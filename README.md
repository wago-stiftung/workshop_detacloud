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
    
    ```

## 
