# REST-API
Using flask and sqlite3 library to import methods.Implementing CRUDE style .

from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

DATABASE = "database.db"

# Create table
def init_db():
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute("""
    CREATE TABLE IF NOT EXISTS records(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER NOT NULL
    )
    """)

    conn.commit()
    conn.close()

init_db()


# Home Route
@app.route("/")
def home():
    return jsonify({"message": "REST API Running"})


# CREATE
@app.route("/records", methods=["POST"])
def create_record():

    data = request.get_json()

    if not data or "name" not in data or "age" not in data:
        return jsonify({"error": "Name and age required"}), 400

    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute(
        "INSERT INTO records(name, age) VALUES(?, ?)",
        (data["name"], data["age"])
    )

    conn.commit()

    record_id = cursor.lastrowid

    conn.close()

    return jsonify({
        "message": "Record created",
        "id": record_id
    }), 201


# READ ALL
@app.route("/records", methods=["GET"])
def get_records():

    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM records")

    rows = cursor.fetchall()

    conn.close()

    result = []

    for row in rows:
        result.append({
            "id": row[0],
            "name": row[1],
            "age": row[2]
        })

    return jsonify(result)


# READ ONE
@app.route("/records/<int:id>", methods=["GET"])
def get_record(id):

    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute(
        "SELECT * FROM records WHERE id=?",
        (id,)
    )

    row = cursor.fetchone()

    conn.close()

    if row is None:
        return jsonify({"error": "Record not found"}), 404

    return jsonify({
        "id": row[0],
        "name": row[1],
        "age": row[2]
    })


# UPDATE
@app.route("/records/<int:id>", methods=["PUT"])
def update_record(id):

    data = request.get_json()

    if not data:
        return jsonify({"error": "No data provided"}), 400

    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute(
        "UPDATE records SET name=?, age=? WHERE id=?",
        (data["name"], data["age"], id)
    )

    conn.commit()

    if cursor.rowcount == 0:
        conn.close()
        return jsonify({"error": "Record not found"}), 404

    conn.close()

    return jsonify({"message": "Record updated"})


# DELETE
@app.route("/records/<int:id>", methods=["DELETE"])
def delete_record(id):

    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    cursor.execute(
        "DELETE FROM records WHERE id=?",
        (id,)
    )

    conn.commit()

    if cursor.rowcount == 0:
        conn.close()
        return jsonify({"error": "Record not found"}), 404

    conn.close()

    return jsonify({"message": "Record deleted"})


if __name__ == "__main__":
    app.run(debug=True)
