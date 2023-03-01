<template>
    <div class="">
        <p v-if="isOffline" style="color: red;">You are no offline</p>
    </div>
    <div style="margin: 10px 0px;">
        <input v-model="title" type="text" style="margin-right: 2px;">
        <button @click="this.save()">Save</button>
        <!-- <button @click="this.update()">Update</button> -->
    </div>
    <div class="">
        <input type="file" @change="image">
    </div>
    <hr>
    <h2>All Notes</h2>
    <div v-if="tasks.length > 0" class="">
        <div v-if="tasks.length > 0" style="display: flex; justify-content: space-between; align-items: center;" v-for="(task, index) in tasks" :key="index">
            <p>
                {{task.title}} 
            </p>
            <div class="">
                <button style="margin-right: 2px;" @click="this.delete(task.id)">delete</button>
                <button @click="this.update(task.id)">edit</button>
            </div>
        </div>
        <div v-else class="">
            There is no Tasks
        </div>
        <hr>
        <h2>All Images</h2>
        <div class="" style="display: flex;">
            <div v-if="images.length > 0" class="" style="margin-right: 3px;"  v-for="(img, index) in images" :key="index">
                <img style="width: 50px; height: 50px; border-radius: 50%;" :src="img.imgUrl" alt="">
            </div>
            <div v-else class="">
                There is no Images
            </div>
        </div>
    </div>
    <div v-else class="">
        There is no value.
    </div>
</template>

<script>

    import Localbase from 'localbase'

    let db = new Localbase('pwa')

    export default {
        name: 'App',
        data() {
            return {
                title: null,
                tasks: [],
                images: [],
                isEdit: false,
                isOffline: !navigator.onLine
            }
        },
        mounted(){
            db.collection('tasks').get().then(tasks => {
                this.tasks = tasks
            })
            db.collection('images').get().then(images => {
                this.images = images
            })
            window.addEventListener('offline', function(){
                this.isOffline = true;
            })
            window.addEventListener('online', function(){
                this.isOffline = false;
            })
        },
        methods: {
            image(e){
                let file = e.target.files[0];
                // let file = files[0];
                const reader = new FileReader();
                // let url = reader.readAsDataURL(file);

                reader.onloadend = () => {
                    const base64String = reader.result;
                    console.log(base64String);
                    db.collection('images').add({
                        id: 1,
                        imgUrl: base64String,
                    })
                    //   localStorage.setItem('wallpaper', base64String);
                    //   document.body.style.background = `url(data:image/png;base64,${base64String})`;
                };
                reader.readAsDataURL(file);
                // console.log(base64String);
                // console.log(url);
            },
            getTasks() {
                db.collection('tasks').get();
            },
            save() {
                let date = new Date();
                let task = {
                    id: this.tasks.length,
                    title: this.title,
                    added_date: date.getTime()
                };
                this.tasks.push(task);
                
                db.collection('tasks').add(task)
            },
            delete(id) {
                console.log(id);
                console.log("hi");
                let deletedTasks = this.tasks.filter(task => task.id != id)
                db.collection('tasks').doc({ id: id }).delete()

                this.tasks = deletedTasks;
            },
            update(id){
                let date = new Date();
                let editedTask = {
                    id: id,
                    title: this.title,
                    added_date: date.getTime()
                };
                db.collection('tasks').doc({ id: id }).update(editedTask)
                let updatedTask = this.tasks.map(task => task.id != id ? task : editedTask);
                console.log(updatedTask);
                this.tasks = updatedTask;
                this.title = "";
            }
        },
       
    }
</script>

<style lang="scss">

</style>