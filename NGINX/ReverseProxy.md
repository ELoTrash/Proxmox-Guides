ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;

# Upstream configuration
upstream plex {
    server YOURPLEXIP:32400;
    keepalive 4;
}

upstream jellyfin {
    server YOURJELLYFINIP:8096;
    keepalive 4;
}

# Main website
server {
    listen 443;
    server_name YOURDOMAIN www.YOURDOMAIN;
    root /var/www/YOURDOMAIN;
    index index.html index.htm index.nginx-debian.html;

    ssl_certificate /etc/ssl/YOURDOMAIN.com.pem;
    ssl_certificate_key /etc/ssl/YOURDOMAIN.com.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
#    add_header Content-Security-Policy "default-src https: data: blob: ; img-src 'self' https://* ; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";
#    add_header Permissions-Policy "accelerometer=(), ambient-light-sensor=(), battery=(), bluetooth=(), camera=(), clipboard-read=(), display-capture=(), document-domain=(), encrypted-media=(), gamepad=(), geolocation=(), gyroscope=(), hid=(), idle-detection=(), interest-cohort=(), keyboard-map=(), local-fonts=(), magnetometer=(), microphone=(), payment=(), publickey-credentials-get=(), serial=(), sync-xhr=(), usb=(), xr-spatial-tracking=()" always;
    add_header X-XSS-Protection "0";

    location / {
        try_files $uri $uri/ =404;
    }
}

# Plex
server {
    listen 443;
    server_name plex.YOURDOMAIN.com;

    ssl_certificate /etc/ssl/YOURDOMAIN.com.pem;
    ssl_certificate_key /etc/ssl/YOURDOMAIN.com.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
#    add_header Content-Security-Policy "default-src https: data: blob: ; img-src 'self' https://* ; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";
    add_header Permissions-Policy "accelerometer=(), ambient-light-sensor=(), battery=(), bluetooth=(), camera=(), clipboard-read=(), display-capture=(), document-domain=(), encrypted-media=(), gamepad=(), geolocation=(), gyroscope=(), hid=(), idle-detection=(), interest-cohort=(), keyboard-map=(), local-fonts=(), magnetometer=(), microphone=(), payment=(), publickey-credentials-get=(), serial=(), sync-xhr=(), usb=(), xr-spatial-tracking=()" always;
    add_header X-XSS-Protection "0";
    
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://plex;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_redirect off;
        proxy_buffering off;
    }
}

# Jellyfin
server {
    listen 443;
    server_name jellyfin.YOURDOMAIN.com;

    ssl_certificate /etc/ssl/YOURDOMAIN.com.pem;
    ssl_certificate_key /etc/ssl/YOURDOMAIN.com.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
#    add_header Content-Security-Policy "default-src https: data: blob: ; img-src 'self' https://* ; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' https://www.gstatic.com https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";
#    add_header Permissions-Policy "accelerometer=(), ambient-light-sensor=(), battery=(), bluetooth=(), camera=(), clipboard-read=(), display-capture=(), document-domain=(), encrypted-media=(), gamepad=(), geolocation=(), gyroscope=(), hid=(), idle-detection=(), interest-cohort=(), keyboard-map=(), local-fonts=(), magnetometer=(), microphone=(), payment=(), publickey-credentials-get=(), serial=(), sync-xhr=(), usb=(), xr-spatial-tracking=()" always;
    add_header Content-Security-Policy "default-src https: data: blob: http://image.tmdb.org; style-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net/npm/jellyskin@latest/dist/main.css; script-src 'self' 'unsafe-inline' https://www.gstatic.com/cv/js/sender/v1/cast_sender.js https://www.youtube.com blob:; worker-src 'self' blob:; connect-src 'self'; object-src 'none'; frame-ancestors 'self'";
    add_header X-XSS-Protection "0";
#    proxy_cache_path /var/cache/nginx/jellyfin levels=1:2 keys_zone=jellyfin:100m max_size=15g inactive=30d use_temp_path=off; 

   # Cache images (inside server block)

   location ~ /Items/(.*)/Images {
        proxy_pass http://jellyfin;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
        # Remove Cache-Control header sent by Jellyfin
        proxy_hide_header Cache-Control;
        proxy_cache_valid 200 206 301 302 30d;
        proxy_ignore_headers Expires Cache-Control Set-Cookie X-Accel-Expires;
        proxy_http_version 1.1;
        # Force Cache-Control header to cache content
        add_header Cache-Control "public, max-age=3600";  # Cache for 1 hour
        proxy_cache jellyfin;
        proxy_cache_revalidate on;
        proxy_cache_lock on;
        add_header X-Cache-Status $upstream_cache_status;
}
   location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://jellyfin;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_redirect off;
        proxy_buffering off;
    }

}



####################### Example caching video from jelly:
# Must be in HTTP block
# Set in-memory cache-metadata size in keys_zone, size of video caching and how many days a cached object should persist
proxy_cache_path  /var/cache/nginx/jellyfin-videos levels=1:2 keys_zone=jellyfin-videos:100m inactive=90d max_size=35000m;
map $request_uri $h264Level { ~(h264-level=)(.+?)& $2; }
map $request_uri $h264Profile { ~(h264-profile=)(.+?)& $2; }

# Set in Server block
location ~* ^/Videos/(.*)/(?!live)
{
  # Set size of a slice (this amount will be always requested from the backend by nginx)
  # Higher value means more latency, lower more overhead
  # This size is independent of the size clients/browsers can request
  slice 2m;

  proxy_cache jellyfin-videos;
  proxy_cache_valid 200 206 301 302 30d;
  proxy_ignore_headers Expires Cache-Control Set-Cookie X-Accel-Expires;
  proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
  proxy_connect_timeout 15s;
  proxy_http_version 1.1;
  proxy_set_header Connection "";
  # Transmit slice range to the backend
  proxy_set_header Range $slice_range;

  # This saves bandwidth between the proxy and jellyfin, as a file is only downloaded one time instead of multiple times when multiple clients want to at the same time
  # The first client will trigger the download, the other clients will have to wait until the slice is cached
  # Esp. practical during SyncPlay
  proxy_cache_lock on;
  proxy_cache_lock_age 60s;

  proxy_pass http://$jellyfin:8096;
  proxy_cache_key "jellyvideo$uri?MediaSourceId=$arg_MediaSourceId&VideoCodec=$arg_VideoCodec&AudioCodec=$arg_AudioCodec&AudioStreamIndex=$arg_AudioStreamIndex&VideoBitrate=$arg_VideoBitrate&AudioBitrate=$arg_AudioBitrate&SubtitleMethod=$arg_SubtitleMethod&TranscodingMaxAudioChannels=$arg_TranscodingMaxAudioChannels&RequireAvc=$arg_RequireAvc&SegmentContainer=$arg_SegmentContainer&MinSegments=$arg_MinSegments&BreakOnNonKeyFrames=$arg_BreakOnNonKeyFrames&h264-profile=$h264Profile&h264-level=$h264Level&slicerange=$slice_range";
  
  # add_header X-Cache-Status $upstream_cache_status; # This is only for debugging cache

}