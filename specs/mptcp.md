# MPTCP

A single transport session as seen by the application layer might be striped or otherwise multiplexed across 
multiple IP layer paths between the session’s two endpoints. 

An over-arching expectation is that TCP-based applications see the traditional TCP API, 
but gain performance beneﬁts when their session transparently utilises multiple, potentially divergent network layer paths. 

These beneﬁts include being able to stripe data over parallel paths for additional speed (where multiple similar paths exist concurrently), or seamlessly maintaining TCP sessions when an individual path fails (or becomes too ‘expensive’) or as a mobile device’s multiple underlying network interfaces come and go. 
