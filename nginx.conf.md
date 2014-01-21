events {
	worker_connections 1024;
}

http {
	server {
		listen 80;
		server_name localhost;

		location / {
			fastcgi_pass   localhost:8000;

			fastcgi_param   SCRIPT_NAME             $fastcgi_script_name;
			fastcgi_param   REQUEST_METHOD          $request_method;
			fastcgi_param	CONTENT_LENGTH		$content_length;
			fastcgi_param	HTTP_RANGE		$http_range;
			fastcgi_param	DOCUMENT_URI		$document_uri;
			fastcgi_param	REQUEST_URI		$request_uri;
		}
	}

	client_max_body_size 0;
}
