# expose-localhost-remotely

These bash functions and config edits work together to securely
**serve your localhost over the internet**
using SSH remote port forwarding. 
 
Note: this method requires two remote users and root access to a remote server with port 80 open, however:
- Using root is only required in this example because root is 
[required](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)
to bind to port 80. If you don't mind serving on a different port (e.g., 8080), 
non-root users can bind to ports above the 0-1023 range. 
- Using two separate users on the remote server 
(as well as editing the .bashrc files for those users)
is only required in this example because the port I want to bind to (80)
is usually occupied by an apache web server. If the binding port isn't 
already occupied, this method can be modified to require only one user. 




## Local Machine  
### .bashrc functions
```bash
function rediron() {
	tmux kill-session -t redir;
	tmux new-session -t redir -d;
	tmux send-keys 'tmux rename-window "redir"' C-m; 
	tmux send-keys 'ssh awsredirect' C-m; # first to kill apache
	sleep 1;
	tmux send-keys 'exit' C-m; 
	sleep 2;
	tmux send-keys 'ssh awsredirect' C-m; # second to start forwarding
}


function rediroff() {
	tmux kill-session -t awsredir;
	tmux new-session -t foo -d;
	tmux send-keys 'ssh aws' C-m; # turn apache back on
	sleep 2;
	tmux send-keys 'exit' C-m;
	tmux kill-session -t foo;
}
```

### ssh_config entries
```bash
Host awsredirect
    Hostname  <serverip>
    User root
    Port 22
    IdentityFile <path-to-pem>
    IdentitiesOnly yes 
    RemoteForward 80 127.0.0.1:8000  # change 8000 to port you want to serve.
    			  	      # it will still forward to server's port 80                     
Host aws
    Hostname  <serverip>
    User ubuntu
    Port 22
    IdentityFile <path/to/key>
    IdentitiesOnly yes 
```



##  Remote Server
### sshd_config
`GatewayPorts yes #allow remote port forwarding`

### root user's .bashrc
```bash
if [[-n $SSH_CONNECTION]] ; then
	service apache2 stop    #  stop apache so the tunnel can be served on port 80
fi
```

### standard user's .bashrc
```bash
if [[-n $SSH_CONNECTION]] ; then
	service apache2 start   # re-start apache after tunnel closes
fi
```



