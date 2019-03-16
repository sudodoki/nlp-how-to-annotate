FROM parentheticalenterprises/sbcl-quicklisp-base

RUN apt-get update && apt-get install -y git libyaml-dev gcc locales && apt-get clean

RUN locale-gen en_US.UTF-8  
ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en  
ENV LC_ALL en_US.UTF-8  

# Used to update builds whenever github repo gets update https://stackoverflow.com/a/49772666/1976857
ARG CACHEBUST
RUN git clone --single-branch --branch master https://github.com/sudodoki/ann.git ann

COPY users.txt ann/users.txt
WORKDIR ann

EXPOSE 7001

RUN echo "(push :dev *features*)\n$(cat run.lisp)" > run.lisp

ENTRYPOINT ["sbcl", "--load", "hunch.lisp", "--load", "run.lisp"]
# ENTRYPOINT ["sbcl", "--noinform", "--disable-ldb", "--lose-on-corruption", "--disable-debugger", "--load", "hunch.lisp", "--load", "run.lisp"]
