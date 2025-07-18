from flask import Flask, request, render_template_string, redirect, url_for
import os

app = Flask(__name__)

UPLOAD_FOLDER = "cloud"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

HTML_PAGE = """
<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<title>Cloud Mamie</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #fdf6e3;
    color: #333;
    max-width: 500px;
    margin: 30px auto;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px #ccc;
  }
  h1 {
    text-align: center;
    color: #6b4f4f;
  }
  form {
    display: flex;
    flex-direction: column;
    gap: 15px;
  }
  input[type="file"] {
    font-size: 1.1em;
  }
  input[type="submit"] {
    background-color: #a57c7c;
    color: white;
    border: none;
    padding: 12px;
    border-radius: 8px;
    font-size: 1.2em;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  input[type="submit"]:hover {
    background-color: #805353;
  }
  ul {
    list-style: none;
    padding-left: 0;
  }
  li {
    margin-bottom: 8px;
  }
  a {
    text-decoration: none;
    color: #805353;
    font-weight: bold;
  }
  a:hover {
    text-decoration: underline;
  }
  .message {
    padding: 10px;
    background-color: #d6f0d6;
    border: 1px solid #6ca96c;
    border-radius: 5px;
    color: #2f662f;
    margin-bottom: 20px;
    text-align: center;
    font-weight: bold;
  }
</style>
</head>
<body>
  <h1>Bienvenue sur le Cloud Mamie</h1>

  {% if message %}
    <div class="message">{{ message }}</div>
  {% endif %}

  <form method="post" enctype="multipart/form-data">
    <input type="file" name="files" multiple required>
    <input type="submit" value="Envoyer">
  </form>

  <h2>Fichiers disponibles :</h2>
  <ul>
    {% for filename in files %}
      <li><a href="/cloud/{{ filename }}">{{ filename }}</a></li>
    {% else %}
      <li>Aucun fichier pour le moment.</li>
    {% endfor %}
  </ul>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def upload_and_list():
    message = ""
    if request.method == "POST":
        uploaded_files = request.files.getlist("files")
        saved_files = []
        for file in uploaded_files:
            if file and file.filename:
                filepath = os.path.join(UPLOAD_FOLDER, file.filename)
                file.save(filepath)
                saved_files.append(file.filename)
        if saved_files:
            message = f"Fichier(s) envoyé(s) avec succès : {', '.join(saved_files)}"
        else:
            message = "Aucun fichier sélectionné."
        # Après l'upload, on recharge la page en GET pour éviter les re-soumissions
        files = os.listdir(UPLOAD_FOLDER)
        return render_template_string(HTML_PAGE, files=files, message=message)

    files = os.listdir(UPLOAD_FOLDER)
    return render_template_string(HTML_PAGE, files=files, message=message)

@app.route("/cloud/<path:filename>")
def uploaded_file(filename):
    filepath = os.path.join(UPLOAD_FOLDER, filename)
    if not os.path.isfile(filepath):
        return "Fichier non trouvé", 404
    return app.send_static_file(os.path.join("cloud", filename))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
