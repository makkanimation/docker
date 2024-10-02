![image](https://github.com/user-attachments/assets/09cba3a1-534f-429d-813b-f31796a9367d)

# Which base model we need to use either node
FROM node:20
# Create directory inside docker
WORKDIR /myapp 
# copy working directory inside myapp folder
either "COPY . ." or  
"COPY . /myapp"
# get node_module folder
RUN npm install
# run on particular port
EXPOSE 3000
# after creating image run command install terminal
CMD ["npm","start"]
