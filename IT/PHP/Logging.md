Hey guys, I want to share my findings, finally made it work, maybe it will be helpful for somebody  
  
The primary problem was related to the configuration of the PHP core itself. I had `error_log = syslog` defined in my `php.ini`, I also did try other options too. It's kind of a weird approach, but to make it work, we need to unset `error_log =` to activate the _SAPI error logger_, so php-fpm can actually send errors with **fascgi protocol**. Looks like you can **not** have both, e.g., `syslog` and `SAPI` working at the same time. The weird part is it is impossible to specify using SAPI something like `error_log = SAPI`; it won't work. There are two options:  
1. comment `error_log =` in `php.ini`  
2. use `php_admin_value[error_log] = none` in `php-fpm.conf` to clear a previously set value  
  
Now, obviously, `fastcgi.logging` should be turned on. While I was playing with various configurations, I did disable it and completely forgot to enable it back. I have `php_admin_flag[fastcgi.logging] = On` in my `php-fpm.conf` .  
  
Other stuff good to know:  
We can safely turn off `catch_workers_output` since, as I mentioned before, this does not affect anything.