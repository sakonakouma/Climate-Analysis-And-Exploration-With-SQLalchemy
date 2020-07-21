import numpy as np

import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func


from flask import Flask, jsonify

engine = create_engine("sqlite:///hawaii.sqlite")

Base = automap_base()

Base.prepare(engine, reflect = True)

Measurement = Base.classes.measurement
Station = Base.classes.station

app = Flask(__name__)

@app.route("/")
def home():
    return (
        f"Welcome to my Precipitation API home page"
        f"Available Route:<br/>"
        f"/api/v1.0/precipitation<br/>"
        f"/api/v1.0/stations<br/>"
        f"/api/v1.0/tobs<br/>"
        f"/api/v1.0/<start><br/>"
        f"/api/v1.0/<start>/<end><br/>")

@app.route("/api/v1.0/precipitation")
def precipitation():
    session = Session(engine)
    
    results = session.query(Measurement.date, Measurement.prcp).\
    filter(Measurement.date >= "2016-08-22").\
    filter(Measurement.date <= "2017-08-23").\
    order_by(Measurement.date)
    
    precip_data = []
    for r in results:
        precip_dict = {}
        precip_dict['date'] = r.date
        precip_dict['prcp'] = r.prcp
        precip_data.append(precip_dict)
    
    return jsonify(precip_data)

    session.close()

@app.route("/api/v1.0/stations")
def stations():
    session = Session(engine)
    
    results = session.query(Station.name).all()
    
    session.close()
    
    station_names = list(np.ravel(results[1:9]))
    
    return jsonify(station_names)

@app.route("/api/v1.0/tobs")
def temperatures():
    session = Session(engine)
         
    temps = session.query(Measurement.date, Measurement.tobs).\
    filter(Measurement.date >= "2016-08-22").\
    filter(Measurement.date <= "2017-08-23").\
    order_by(Measurement.date).all()
    
    session.close()
    
    temps_data = []
    for t in temps:
        temps_dict = {}
        temps_dict['date'] = t.date
        temps_dict['tobs'] = t.tobs
        temps_data.append(temps_dict)
    
    return jsonify(temps_data)

@app.route("/api/v1.0/<start>")
def temp_per_date(start):
    session = Session(engine)
    
    results = session.query(func.min(Measurement.tobs).label('min'),\
    func.avg(Measurement.tobs).label('avg'), func.max(Measurement.tobs).label('max')).\
    filter(Measurement.date >= start).all()
    
    session.close()
    
    temp_per_date_data = []
    for r in results:
        temp_per_date_dict = {}
        temp_per_date_dict['Start Date'] = start
        temp_per_date_dict['Min Temp'] = r.min
        temp_per_date_dict['Avg Temp'] = r.avg
        temp_per_date_dict['Max Temp'] = r.max
        temp_per_date_data.append(temp_per_date_dict)
        
        return jsonify(temp_per_date_data)

@app.route("/api/v1.0/<start>/<end>")
def temp_start_end(start, end):
    session.Session(engine)
    
    results = session.query(func.min(Measurement.tobs).label('min'),\
    func.avg(Measurement.tobs).label('avg'), func.max(Measurement.tobs).label('max')).\
    filter(Measurement.date >= start).\
    filter(Measurement.date <= end).all()
    
    session.close()
    
    temp_start_end_data = []
    for r in results:
        temp_start_end_dict = {}
        temp_start_end_dict['Start Date'] = start
        temp_start_end_dict['Min Temp'] = r.min
        temp_start_end_dict['Avg Temp'] = r.avg
        temp_start_end_dict['Max Temp'] = r.max
        temp_start_end_dict['End Date'] = end
        temp_start_end_data.append(temp_start_end_dict)
        
        return jsonify(temp_start_end_data)
    

if __name__ == '__main__':
    app.run(debug=True)