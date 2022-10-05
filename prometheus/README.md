start prometheus with docker `docker run -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus`

Check on http://localhost:9090/targets that the targets are loaded