FROM microsoft/aspnetcore-build:2 AS builder
WORKDIR /app

# caches restore result by copying csproj file separately
COPY *.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish --output /app/ --configuration Debug
RUN sed -n 's:.*<AssemblyName>\(.*\)</AssemblyName>.*:\1:p' *.csproj > __assemblyname
RUN if [ ! -s __assemblyname ]; then filename=$(ls *.csproj); echo ${filename%.*} > __assemblyname; fi

# Stage 2
FROM microsoft/aspnetcore:2
WORKDIR /app
COPY --from=builder /app .

RUN apt update

RUN apt install -y openssh-server
RUN echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh && chmod 700 /root/.ssh
COPY id_rsa_vscode.pub /root/.ssh/authorized_keys
RUN chmod 700 /root/.ssh/authorized_keys

RUN apt install -y unzip curl procps
RUN curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v vs2017u5 -l ~/vsdbg

ENV PORT 80
EXPOSE 80

ENTRYPOINT service ssh start && dotnet $(cat /app/__assemblyname).dll
