FROM python

WORKDIR /api_json

COPY ./sql_demo.py .

RUN pip3 install pymysql
RUN pip3 install cryptography

CMD ["python", "sql_demo.py"]