#
# BUILDER 
##############################################################################################################
FROM python:3.10 as builder

ENV BAZEL_VERSION=5.4.0
ENV SAXML_VERSION=1.2.0

RUN curl https://bazel.build/bazel-release.pub.gpg | apt-key --keyring /usr/share/keyrings/bazel-archive-keyring.gpg add - && \
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list && \
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
apt -y update && \
apt install -y \
  apt-transport-https \
  bazel-${BAZEL_VERSION} \
  curl \
  gnupg \
  google-cloud-cli \
  patch \
  python3-pip && \
rm -rf /var/lib/apt/lists/* && \
update-alternatives --install /usr/bin/bazel bazel /usr/bin/bazel-${BAZEL_VERSION} 20

ENV PATH=/venv/bin:${PATH}
COPY requirements-build.txt /venv/requirements-build.txt
RUN python -m venv /venv && \
pip install --no-cache-dir -r /venv/requirements-build.txt

RUN git clone https://github.com/google/saxml.git && \
cd saxml && \
git checkout r${SAXML_VERSION} && \
bazel build saxml/client/python:sax.cc --compile_one_dependency

RUN cp --dereference --recursive /saxml/bazel-out/k8-fastbuild/bin /venv/lib/saxml-client

#
# FINAL 
##############################################################################################################
FROM python:3.10-slim as final

ENV PATH=/venv/bin:${PATH}
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONPATH=/venv/lib/saxml-client/saxml/client/python
ENV PYTHONUNBUFFERED=1

COPY requirements.txt /venv/requirements.txt
RUN python -m venv /venv && \
pip install --no-cache-dir -r /venv/requirements.txt

COPY --from=builder /venv/lib/saxml-client /venv/lib/saxml-client
COPY http_server.py /saxml/saxml/httpserver/

WORKDIR /saxml/saxml/httpserver

EXPOSE 8888

CMD ["uvicorn", "http_server:app", "--host=0.0.0.0", "--port=8888"]
