FROM n8nio/runners:1.121.0

USER root
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas requests
COPY n8n-task-runners.json /etc/n8n-task-runners.json
USER runner