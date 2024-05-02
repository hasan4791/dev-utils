# Custom catalog with multiple operators

A small hack to have multiple operators being served by a single catalog source operator while performing image mirrors using "oc mirror" utility.

## Steps

    1. Create a ImageSetConfiguration with filters for a operator, lets say "Operator-A" 
            ####################################################################
            apiVersion: mirror.openshift.io/v1alpha2
            kind: ImageSetConfiguration
            storageConfig:
            registry:
                imageURL: myregistry/redhat-operator-index:v4.13
                skipTLS: false
            mirror:
            operators:
                - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.13
                packages:
                    - name: Operator-A
            ####################################################################

    2. Run oc-mirror to the destination registry "myregistry"

    3. Extract the "index.json" file from the "/configs" folder of the image "myregistry/redhat-operator-index:v4.13" to some directory using "oc image extract" command.

    4. Repeat the above steps for another operator, lets say "Operator-B"

    5. Merge the contents of "index.json" from Operator-A and Operator-B into a new file called "index.json"

    6. Using the following Dockerfile, create a new custom catalog image which would serve both the operators lifecycle management.
            ####################################################################
            FROM registry.redhat.io/redhat/redhat-operator-index:v4.13
            USER root
            RUN rm -rf /configs /cache
            COPY configs /configs
            RUN /bin/opm serve /configs --cache-dir /cache --cache-only && \
                chown -R 1001:0 /cache
            USER 1001
            CMD ["serve", "/configs", "--cache-dir=/cache"]
            ####################################################################
