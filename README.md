# b44
from flask import Flask, render_template, request
import requests
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)

API_KEY = os.getenv("API_KEY")

@app.route("/", methods=["GET","POST"])
def index():

    data = None

    if request.method == "POST":

        address = request.form["address"]

        url = f"https://api.basescan.org/api?module=account&action=txlist&address={address}&startblock=0&endblock=99999999&sort=asc&apikey={API_KEY}"

        r = requests.get(url).json()

        txs = r["result"]

        incoming = 0
        outgoing = 0

        for tx in txs:

            value = int(tx["value"])/1e18

            if tx["to"] and tx["to"].lower()==address.lower():
                incoming += value

            if tx["from"].lower()==address.lower():
                outgoing += value

        data = {
            "count":len(txs),
            "incoming":round(incoming,4),
            "outgoing":round(outgoing,4),
            "volume":round(incoming+outgoing,4)
        }

    return render_template("index.html",data=data)

if __name__=="__main__":
    app.run(debug=True)
