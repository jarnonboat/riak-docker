FROM centos:7

ENV OS_FAMILY centos
ENV OS_VERSION 7

ENV RIAK_VERSION 2.2.6
ENV RIAK_HOME /usr/lib/riak
ENV RIAK_FLAVOR KV

# install dependencies
RUN yum update -y && \
    yum install -y epel-release && \
    yum install -y wget zip unzip dpkg-devel dpkg-dev automake which curl gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git wxBase.x86_64 pam-devel fop unixODBC

# install erlang
WORKDIR /opt
COPY src/otp_src_R16B02-basho10.tar.gz .
RUN tar zxvf otp_src_R16B02-basho10.tar.gz && \
    cd OTP_R16B02_basho10 && \
    ./otp_build autoconf && \
    CFLAGS="-DOPENSSL_NO_EC=1" ./configure && \
    make && \
    make install

# install riak
WORKDIR /opt
COPY src/riak-setup.zip .
RUN unzip riak-setup.zip && \ 
    cd riak-setup && \
    make rel

RUN mkdir $RIAK_HOME && \
    mv /opt/riak-setup/rel/riak /usr/lib/

# config riak path
COPY rel_to_pkg.sh /tmp
RUN chmod a+x /tmp/rel_to_pkg.sh && /tmp/rel_to_pkg.sh

ENV PATH $RIAK_HOME/bin:$PATH

# Install riak explorer 
RUN curl -sSL https://github.com/basho-labs/riak_explorer/releases/download/1.4.1/riak_explorer-1.4.1.patch-centos-7.tar.gz | tar -zxf - -C $RIAK_HOME --strip-components 2
RUN for f in riak_pb riak_kv riak_ts riak_dt riak_search riak_yokozuna;do rm -f $RIAK_HOME/lib/basho-patches/$f*; done

# Tune Riak configuration settings for the container
RUN sed -i.bak 's/listener.http.internal = 127.0.0.1/listener.http.internal = 0.0.0.0/' /etc/riak/riak.conf && \
    sed -i.bak 's/listener.protobuf.internal = 127.0.0.1/listener.protobuf.internal = 0.0.0.0/' /etc/riak/riak.conf && \
    echo "anti_entropy.concurrency_limit = 1" >> /etc/riak/riak.conf && \
    echo "javascript.map_pool_size = 0" >> /etc/riak/riak.conf && \
    echo "javascript.reduce_pool_size = 0" >> /etc/riak/riak.conf && \
    echo "javascript.hook_pool_size = 0" >> /etc/riak/riak.conf

# Expose default ports
EXPOSE 8087
EXPOSE 8098

# Expose volumes for data and logs
VOLUME /var/log/riak
VOLUME /var/lib/riak

# Install custom start script
COPY riak-cluster.sh $RIAK_HOME/riak-cluster.sh
RUN chmod a+x $RIAK_HOME/riak-cluster.sh

# Install custom hooks
COPY prestart.d /etc/riak/prestart.d
COPY poststart.d /etc/riak/poststart.d

# Prepare for bootstrapping schemas
RUN mkdir -p /etc/riak/schemas

WORKDIR /var/lib/riak

CMD ["/usr/lib/riak/riak-cluster.sh"]

RUN rm -rf /opt/*