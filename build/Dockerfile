FROM dmh2000/ubuntu-gpp

# working directory
WORKDIR /app

# copy the build script to /app
COPY build-dev.sh .

# make it executable in the container
RUN chmod +x build-dev.sh

# create the output directory
RUN mkdir output

# run the build script (only happens when running the container)
CMD ["./build-dev.sh"]
