FROM python:latest

WORKDIR /root

COPY blocklist.txt .
COPY config.yml .
COPY update-zonefile.py .
COPY requirements.txt .

ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

RUN pip install --upgrade pip && \
    pip install -r requirements.txt

RUN mkdir /zones

CMD ["python3", "./update-zonefile.py", "--no-bind", "/zones/rpz-adblock.zone", "rpz.adblock"]
