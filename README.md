from flask import Flask, request, render_template_string, abort, send_from_directory
import os
import mimetypes

app = Flask(__name__)

UPLOAD_FOLDER = "cloud"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

HTML_PAGE = """
<!doctype html>
<title>Cloud Mamie</title>
<h1>Envoyer un fichier</h1>
<form method=post enctype=multipart/form-data>
  <input type=file name=file>
  <input type=submit value=Uploader>
</form>
<h2>Fichiers :</h2>
<ul>
{% for filename in files %}
  <li><a href="/cloud/{{ filename }}">{{ filename }}</a></li>
{% endfor %}
</ul>
"""

CODE_PAGE = """
<!doctype html>
<title>Contenu de {{ filename }}</title>
<h1>Contenu de {{ filename }}</h1>
<pre>{{ content }}</pre>
<a href="/">Retour</a>
"""

@app.route("/", methods=["GET", "POST"])
def upload_and_list():
    if request.method == "POST":
        file = request.files.get("file")
        if file and file.filename:
            filepath = os.path.join(UPLOAD_FOLDER, file.filename)
            file.save(filepath)
            return f"Fichier {file.filename} envoyé avec succès<br><a href='/'>Retour</a>"
        else:
            return "Aucun fichier sélectionné.<br><a href='/'>Retour</a>"
    files = os.listdir(UPLOAD_FOLDER)
    return render_template_string(HTML_PAGE, files=files)

@app.route("/cloud/<path:filename>")
def uploaded_file(filename):
    filepath = os.path.join(UPLOAD_FOLDER, filename)
    if not os.path.isfile(filepath):
        abort(404)
    mime, _ = mimetypes.guess_type(filepath)
    if mime and mime.startswith("text"):
        with open(filepath, "r", encoding="utf-8") as f:
            content = f.read()
        return render_template_string(CODE_PAGE, filename=filename, content=content)
    else:
        return send_from_directory(UPLOAD_FOLDER, filename, as_attachment=True)
