FROM ubuntu:16.04

RUN apt update
RUN apt install build-essential curl -y
RUN apt install file -y
RUN curl -sSf sh.rustup.rs | sh -s -- -y
RUN echo "export PATH=~/.cargo/bin:$PATH" >> ~/.bashrc
ENV USER=root
