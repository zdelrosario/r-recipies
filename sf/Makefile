# ${RSTUDIO_PASS} # Set in environment variables

start:
	./run-rstudio

stop:
	docker stop rstudio

setup:
	docker pull rocker/geospatial
	mkdir data

clean:
	docker rm rstudio
