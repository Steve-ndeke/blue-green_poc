# Use an official Nginx image as the base image
FROM nginx:alpine

# Copy the HTML and CSS files to the Nginx web server's root directory
COPY index.html /usr/share/nginx/html/
COPY styles.css /usr/share/nginx/html/

# Expose port 80 to the outside world
EXPOSE 80

# Start Nginx when the container launches
CMD ["nginx", "-g", "daemon off;"]
