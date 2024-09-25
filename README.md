pip install Flask Flask-SQLAlchemy
/notizen-app
  ├── app.py
  ├── models.py
  ├── database.db (wird automatisch erstellt)
  └── templates/
        └── index.html (optional für die Benutzeroberfläche)
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# Importiere das Modell
from models import Note

# Route: Alle Notizen abrufen
@app.route('/notes', methods=['GET'])
def get_notes():
    notes = Note.query.all()
    return jsonify([note.to_dict() for note in notes]), 200

# Route: Einzelne Notiz abrufen
@app.route('/notes/<int:id>', methods=['GET'])
def get_note(id):
    note = Note.query.get(id)
    if note:
        return jsonify(note.to_dict()), 200
    else:
        return jsonify({'message': 'Note not found'}), 404

# Route: Notiz erstellen
@app.route('/notes', methods=['POST'])
def create_note():
    data = request.get_json()
    new_note = Note(title=data['title'], content=data['content'])
    db.session.add(new_note)
    db.session.commit()
    return jsonify(new_note.to_dict()), 201

# Route: Notiz aktualisieren
@app.route('/notes/<int:id>', methods=['PUT'])
def update_note(id):
    note = Note.query.get(id)
    if note:
        data = request.get_json()
        note.title = data.get('title', note.title)
        note.content = data.get('content', note.content)
        db.session.commit()
        return jsonify(note.to_dict()), 200
    else:
        return jsonify({'message': 'Note not found'}), 404

# Route: Notiz löschen
@app.route('/notes/<int:id>', methods=['DELETE'])
def delete_note(id):
    note = Note.query.get(id)
    if note:
        db.session.delete(note)
        db.session.commit()
        return jsonify({'message': 'Note deleted'}), 200
    else:
        return jsonify({'message': 'Note not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
from app import db

class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)

    def to_dict(self):
        return {
            'id': self.id,
            'title': self.title,
            'content': self.content
        }

# Datenbank initialisieren
if __name__ == '__main__':
    db.create_all()
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Notizen App</title>
</head>
<body>
    <h1>Notizen</h1>
    <div id="notes-container"></div>

    <script>
        async function fetchNotes() {
            const response = await fetch('/notes');
            const notes = await response.json();
            const container = document.getElementById('notes-container');
            container.innerHTML = '';
            notes.forEach(note => {
                container.innerHTML += `<h2>${note.title}</h2><p>${note.content}</p>`;
            });
        }

        fetchNotes();
    </script>
</body>
</html>
