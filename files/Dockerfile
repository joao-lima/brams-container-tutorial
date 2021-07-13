# OS
FROM ubuntu:20.10

# Install Fortran & C Compilers and autoconf
ENV DEBIAN_FRONTEND=noninteractive
RUN apt update &&\
      apt install -y build-essential gfortran m4 dnsutils openssh-server curl libcurl4 libcurl4-openssl-dev &&\
      apt clean

# Install MPI Libraries and Software
RUN cd /root && curl -sL https://www.mpich.org//static/downloads/3.3.2/mpich-3.3.2.tar.gz | tar -xzf - &&\
  cd mpich-3.3.2 && \
	./configure -disable-fast CFLAGS=-O2 FFLAGS='-w -fallow-argument-mismatch -O2' CXXFLAGS=-O2 FCFLAGS=-O2 -prefix=/opt/mpich3 CC=gcc FC=gfortran F77=gfortran &&\
  make && make install && \
  rm -rf /root/*

# Install WGRIB2 package from NCEP
RUN cd /root && curl https://www.ftp.cpc.ncep.noaa.gov/wd51we/wgrib2/wgrib2.tgz | tar -xzf - &&\
  cd grib2 && \
	sed -i 's/USE_NETCDF3=.*/USE_NETCDF3=0/' makefile && \
	sed -i 's/USE_NETCDF4=.*/USE_NETCDF4=0/' makefile && \
	sed -i 's/USE_REGEX=.*/USE_REGEX=1/' makefile && \
	sed -i 's/USE_TIGGE=.*/USE_TIGGE=1/' makefile && \
	sed -i 's/USE_MYSQL=.*/USE_MYSQL=0/' makefile && \
	sed -i 's/USE_IPOLATES=.*/USE_IPOLATES=3/' makefile && \
	sed -i 's/USE_SPECTRAL=.*/USE_SPECTRAL=0/' makefile && \
	sed -i 's/USE_UDF=.*/USE_UDF=0/' makefile && \
	sed -i 's/USE_OPENMP=.*/USE_OPENMP=0/' makefile && \
	sed -i 's/USE_PROJ4=.*/USE_PROJ4=0/' makefile && \
	sed -i 's/USE_WMO_VALIDATION=.*/USE_WMO_VALIDATION=0/' makefile && \
	sed -i 's/DISABLE_TIMEZONE=.*/DISABLE_TIMEZONE=0/' makefile && \
	sed -i 's/MAKE_FTN_API=.*/MAKE_FTN_API=1/' makefile && \
	sed -i 's/DISABLE_ALARM=.*/DISABLE_ALARM=0/' makefile && \
	sed -i 's/USE_G2CLIB=.*/USE_G2CLIB=0/' makefile && \
	sed -i 's/USE_PNG=.*/USE_PNG=0/' makefile && \
	sed -i 's/USE_JASPER=.*/USE_JASPER=0/' makefile && \
	sed -i 's/USE_AEC=.*/USE_AEC=0/' makefile && \
	make CC=gcc FC=gfortran && make CC=gcc FC=gfortran lib && cd /root && mv grib2 /opt && \
  rm -rf /root/*

# install HDF5              
RUN cd /root &&\
  curl -s https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.12/hdf5-1.12.1/src/hdf5-1.12.1.tar.gz | tar -xzf - &&\
  cd hdf5-1.12.1 && \
  FC=gfortran CC=gcc ./configure --prefix=/opt/hdf5 --enable-fortran &&\
  make && make install &&\
  rm -rf /root/*

# Install the NetCDF C libraries and packages
RUN cd /root && curl -s https://www.unidata.ucar.edu/downloads/netcdf/ftp/netcdf-c-4.7.3.tar.gz | tar -xzf - &&\
  cd netcdf-c-4.7.3 && \
 ./configure --prefix=/opt/netcdfc --disable-dap --disable-netcdf-4 FC=/opt/mpich3/bin/mpif90 CC=/opt/mpich3/bin/mpicc LT_SYS_LIBRARY_PATH=/opt/hdf5/lib CFLAGS='-I/opt/hdf5/include' LIBS='-L/opt/hdf5/lib' &&\
	make && make install &&\
  rm -rf /root/*

# Install the NetCDF Fortran libraries and packages
RUN cd /root && curl -s https://www.unidata.ucar.edu/downloads/netcdf/ftp/netcdf-fortran-4.5.2.tar.gz | tar -xzf - && \
	cd netcdf-fortran-4.5.2 &&\
  export LD_LIBRARY_PATH=/opt/netcdfc/lib:/opt/hdf5/lib:/usr/local/lib &&\
  FC=/opt/mpich3/bin/mpif90 CC=/opt/mpich3/bin/mpicc ./configure --prefix=/opt/netcdff \
    CPPFLAGS='-I/opt/netcdfc/include' LDFLAGS='-L/opt/netcdfc/lib' &&\
  make && make install &&\
  rm -rf /root/*

# Configure ssh (multi-host)
COPY sshconfig /root/.ssh/config
RUN mkdir -p /var/run/sshd \
    && ssh-keygen -A \
    && sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#RSAAuthentication yes/RSAAuthentication yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config \
    && ssh-keygen -f /root/.ssh/id_rsa -t rsa -N '' \
    && chmod 600 /root/.ssh/config \
    && chmod 700 /root/.ssh \
    && cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys 
RUN echo 'root:brams' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# Making BRAMS Model
# needs gcc-10 because of flag -fallow-argument-mismatch
RUN cd /root &&\
  curl http://ftp.cptec.inpe.br/pesquisa/bramsrd/BRAMS/releases/stable/brams-5.6.1.xz | tar -xJf - &&\
  cd BRAMS-5.6.1 &&\
  cd build && \
  ./configure --program-prefix=BRAMS_5.4 --prefix=/root --enable-jules \
	   --with-chem=RELACS_TUV --with-aer=SIMPLE --with-fpcomp=/opt/mpich3/bin/mpif90 \
	   --with-cpcomp=/opt/mpich3/bin/mpicc --with-fcomp=gfortran --with-ccomp=gcc &&\
  make && make install &&\
  rm -rf /root/BRAMS-5.6.1

# Copy script to container
COPY run-brams.sh /root/run-brams

# Clean up unnecessary files and give execution permission to script
RUN chmod +x /root/run-brams

#
WORKDIR /root
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
