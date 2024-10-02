![image](https://github.com/user-attachments/assets/c18cc49b-eb72-41a9-95f7-9623f0139fac)

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
