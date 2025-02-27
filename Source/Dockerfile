FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 9001

# Default ASPNETCORE_ENVIRONMENT to Release
ENV ASPNETCORE_ENVIRONMENT=Release 

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

COPY . ./

FROM build AS publish

COPY ./CDR.DataRecipient.Web/. /app/CDR.DataRecipient.Web
COPY ./CDR.DataRecipient.Repository.SQL/. /app/CDR.DataRecipient.Repository.SQL
COPY ./CDR.DataRecipient/. /app/CDR.DataRecipient
COPY ./CDR.DataRecipient.SDK/. /app/CDR.DataRecipient.SDK
COPY ./CDR.DataRecipient.API.Logger/. /app/CDR.DataRecipient.API.Logger

WORKDIR /app/CDR.DataRecipient.Web
RUN dotnet publish -c Release -o /app/publish/web

COPY supervisord.conf /app/publish/supervisord.conf

FROM base AS final
WORKDIR /app

COPY --from=publish /app/publish/supervisord.conf .
COPY --from=publish /app/publish/web ./web

RUN apt-get update && apt-get install -y supervisor
RUN apt-get update && apt-get install -y sudo

RUN sudo cp ./web/Certificates/ca.crt /usr/local/share/ca-certificates/ca.crt
RUN sudo update-ca-certificates

RUN addgroup --group appgroup --gid 2000 \
&& adduser \    
    --uid 1000 \
    --gid 2000 \
    "appuser" 

RUN chown -R appuser:appgroup /app
RUN chown -R appuser:appgroup /usr/bin/
RUN chown -R appuser:appgroup /tmp
USER appuser:appgroup 

ENV ASPNETCORE_URLS=https://+:9001
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/app/supervisord.conf", "-u", "1000"]