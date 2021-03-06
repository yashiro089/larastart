<template>
  <div class="container">
    <div class="row mt-5" v-if="$gate.isAdmin()">
      <div class="col-md-12">
        <div class="card">
          <div class="card-header">
            <h3 class="card-title">User Management
            </h3>
            <div class="card-tools">
              <button class="btn btn-success" @click="createModal">
                Add New
                <i class="fas fa-user-plus fa-fw">
                </i>
              </button>
            </div>
          </div>
          <!-- /.card-header -->
          <div class="card-body table-responsive p-0">
            <table class="table table-hover">
              <tbody>
                <tr>
                  <th>ID</th>
                  <th>Name</th>
                  <th>Email</th>
                  <th>Type</th>
                  <th>Registered At</th>
                  <th>Modify</th>
                </tr>
                <tr v-for="(user) in users" :key="user.id">
                  <td>{{user.id}}</td>
                  <td>{{user.name}}</td>
                  <td>{{user.email}}</td>
                  <td>{{user.type | upText }}</td>
                  <td>{{user.created_at | newDate }}</td>
                  <td>
                    <a href="#" @click="editModal(user)">
                      <i class="fa fa-edit blue">
                      </i>
                    </a>
                    <a href="#" @click="deleteUser(user.id)">
                      <i class="fa fa-trash red">
                      </i>
                    </a>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <!-- /.card-body -->
        </div>
        <!-- /.card -->
      </div>
    </div>
    <!-- Modal -->
    <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" v-show="editMode" id="addNewLabel">Update User
            </h5>
            <h5 class="modal-title" v-show="!editMode" id="addNewLabel">Add New User
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;
              </span>
            </button>
          </div>
          <form @submit.prevent="editMode ? updateUser() : createUser()">
            <div class="modal-body">
              <div class="form-group">
                <input v-model="form.name" type="text" name="name" placeholder="Name"
                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                <has-error :form="form" field="name">
                </has-error>
              </div>
              <div class="form-group">
                <input v-model="form.email" type="email" name="email" placeholder="Email address"
                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                <has-error :form="form" field="email">
                </has-error>
              </div>
              <div class="form-group">
                <input v-model="form.password" type="password" name="password" placeholder="Password"
                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                <has-error :form="form" field="password">
                </has-error>
              </div>
              <div class="form-group">
                <textarea v-model="form.bio" type="text" name="bio" placeholder="Bio"
                          class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                </textarea>
                <has-error :form="form" field="bio">
                </has-error>
              </div>
              <div class="form-group">
                <select v-model="form.type" type="text" id="type" name="type" placeholder="User Type"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                  <option value="">Select User Role</option>
                  <option value="admin">Admin</option>
                  <option value="user">User</option>
                  <option value="author">Author</option>
                </select>
                <has-error :form="form" field="type">
                </has-error>
              </div>
              <div class="form-group">
                <input v-model="form.photo" type="text" name="photo" placeholder="Photo"
                       class="form-control" :class="{ 'is-invalid': form.errors.has('photo') }">
                <has-error :form="form" field="photo">
                </has-error>
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
              <button v-show = "editMode" type="submit" class="btn btn-success">Update</button>
              <button v-show = "!editMode" type="submit" class="btn btn-primary">Create</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    data(){
      return{
        editMode: true,
        users: {
        }
        ,
        form: new Form({
          id:'',
          name: '',
          email: '',
          password: '',
          type: '',
          bio: '',
          photo: ''
        }
                      )
      }
    }
    ,
    methods:{
      updateUser(){
        this.$Progress.start();
        this.form.put('api/user/' + this.form.id)
          .then(() => {
          //success
          $('#addNew').modal('hide');
          swal(
            'Updated!',
            'User has been updated.',
            'success'
          )
          this.$Progress.finish();
          Fire.$emit('fireLoadUsers');
        })
          .catch(() => {
          this.$Progress.fail();
        });
      },
      createModal(){
        this.editMode = false;
        this.form.reset();
        $('#addNew').modal('show');
      },
      editModal(user){
        this.editMode = true;
        this.form.reset();
        $('#addNew').modal('show');
        this.form.fill(user);
      },
      deleteUser(id){
        swal({
          title: 'Are you sure you want to delete this user?',
          text: "You won't be able to revert this!",
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes, delete it!'
        })
        .then((result) => {
          //Send request to the server
          if(result.value){
            this.form.delete('api/user/' + id)
            .then(() => {
              swal(
                'Deleted!',
                'User has been deleted.',
                'success'
              )
              Fire.$emit('fireLoadUsers');
            })
            .catch(() => {
              swal("Failed", "Cannot continue. Something was wrong");
            });
          }
        })
      },
      loadUsers(){
        if(this.$gate.isAdmin()){
            axios.get("api/user").then(({data }) => (this.users = data.data));
        }
      },
      createUser(){
        this.$Progress.start();
        this.form.post('api/user')
          .then(() => {
          Fire.$emit('fireLoadUsers');
          $('#addNew').modal('hide');
          toast({
            type: 'success',
            title: 'Signed in successfully'
          })
          this.$Progress.finish();
        })
          .catch(() => {
        });
      }
    }
    ,
    created() {
      this.loadUsers();
      // setInterval(function(){this.loadUsers()}, 3000)
      // setInterval(() => this.loadUsers(), 3000);
      Fire.$on('fireLoadUsers',() => {
        this.loadUsers();
      }
              );
    }
  }
</script>
