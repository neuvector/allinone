ARG VERSION=v1.0-slsa
FROM holyspectral/controller:$(VERSION) AS controller
FROM holyspectral/enforcer:$(VERSION) AS enforcer
FROM holyspectral/manager:$(VERSION) AS manager
FROM registry.suse.com/bci/bci-micro:15.6 AS builder

COPY --from=controller / /controller
COPY --from=enforcer / /enforcer
COPY --from=manager / /manager
RUN mkdir -p /chroot && cp -rn /controller/* /chroot && cp -rn /enforcer/* /chroot && cp -rn /manager/* /chroot && rm -rf /controller /enforcer /manager

FROM registry.suse.com/bci/bci-micro:15.6

COPY --from=builder /chroot /

RUN ln -s /usr/bin/python3.12 /usr/bin/python && \                      
    ln -s /usr/bin/python3.12 /usr/bin/python3  && \                    
    pip3 install --no-cache-dir "supervisor==4.2.5" && \                
    pip3 install --no-cache-dir -r /requirements.txt && \               
    ln -s /usr/local/bin/supervisord /usr/bin/supervisord && \          
    rm -r /root/.cache /requirements.txt            

ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
