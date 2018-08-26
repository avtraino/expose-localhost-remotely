# expose-localhost-remotely



 These bash functions and config edits work together
 to serve your localhost over the internet using
 remote port forwarding. This requires only your 
 local machine and a remote machine with port 80 open.



## Local Machine  
### .bashrc functions
```
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

## ssh_config entries
```
Host awsredirect
    Hostname  <serverip>
    User root
    Port 22
    IdentityFile <path-to-pem>
    IdentitiesOnly yes 
    RemoteForward 80 127.0.0.1:8000  # change `8000` to port you want to serve.
    			  	      # it will still forward to server's port 80                     
Host aws
    Hostname  <serverip>
    User ubuntu
    Port 22
    IdentityFile <path/to/key>
    IdentitiesOnly yes 
```


#  Remote Machine
## sshd_config
`GatewayPorts yes #allow remote port forwarding`

## root user's .bashrc
```
if [[-n $SSH_CONNECTION]] ; then
	service apache2 stop    #  stop apache so the tunnel can be served on port 80
fi
```

## standard user's .bashrc
```
if [[-n $SSH_CONNECTION]] ; then
	service apache2 start   # re-start apache after tunnel closes
fi
```
