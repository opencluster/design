# Security Scanning

## Scanning for stale containers.

The system can alert the administrators or generate reports on containers in their environment that have not been updated.

When people create a container... say an HAPROXY container and have it running, then it is tempting to leave it running forever.  
We need to ensure that the container itself is rebuilt and the latest bug fixes and patches have been applied to libraries and the software running in the container.

Trellis should assist in keeping track of this information, and alerting administrators when action needs to be taken.

