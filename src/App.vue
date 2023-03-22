
<template>
  <div class="container py-5 px-md-5" id="app">
    <hr />
    <div class="row mb-3 text-center">
      <div class="col-12">
        <button v-bind:class="{disabled: this.cart.length <= 0}" class="card-link btn btn-dark btn-sm px-3 text-center" @click="showCheckout"><i
            class="fas fa-arrow-right mx-2"></i>Go
          to {{ this.titles }} {{
            cartItemCount
          }}</button>
      </div>
    </div>

    <!-- main -->
    <div class="row pt-2">
      <main class="d-flex justify-content-center">
        <component :is="currentView" :lessonList="lessons" :baseURL="baseURL" @add-to-cart="addToCart"
          @remove-from-cart="removeFromCart" :cart="cart">
        </component>
      </main>
    </div>
  </div>
</template>
<script>
import LessonComponent from "./components/Lesson.vue";
import CheckoutComponent from "./components/Checkout.vue";
export default {
  name: "App",
  data() {
    return {
      currentView: LessonComponent,
      lessons: [],
      cart: [],
      title: 'Cart',
      sortBy: 'subject',  // default sorting value
      orderBy: 'ascending', // default order value
      name: '',
      search: '',
      phoneNumber: '',
      baseURL: 'https://webstoreapp-env.eba-yi2fch33.eu-west-2.elasticbeanstalk.com'
    }
  },
  components: { LessonComponent, CheckoutComponent },
  // fetching the lessons in json from the get path
  created: function () {
    // using promise and fetch to get a list of lessons
    this.getLessons();
  },
  methods: {
    /// returns a new promise that will be resolved or rejected based on the result of the fetch call.
    getLessons() {
      let webstore = this
      fetch(`${this.baseURL}/collections/lessons`).then(
        function (response) {
          response.json().then(
            function (json) {
              // pushing lessons in json format into the lessons array
              webstore.lessons = json;
            }
          )
        });
    },

    // adds a lesson to cart
    addToCart(_id) {
      // find selected lesson id
      var lesson = this.getLessonById(_id);
      if (lesson.space > 0) {
        // decrease lesson space
        --lesson.space;

        // get existing item from cart
        var itemInCart = this.getCartItemFromCartByLessonId(_id);

        if (itemInCart != null) {
          // update existing item in cart with put
          ++itemInCart.space;

        } else {
          // adding new item to cart
          itemInCart = { lessonId: _id, space: 1, lesson: lesson };
          this.cart.push(itemInCart);
        }
      }
    },
    // removes a lesson from cart
    removeFromCart(lessonId) {
      // find selected lesson in cart
      var itemInCart = this.getCartItemFromCartByLessonId(lessonId);

      if (itemInCart.space == 1) {
        // if just one item space is left, remove item completely from cart
        var index = this.cart.map(x => x.lessonId).indexOf(lessonId);
        this.cart.splice(index, 1);

        // redirect user back to home if cart is empty
        if (this.cart.length <= 0) {
          this.showCheckout();
        }
      } else {
        // reduce number of space of item in cart
        --itemInCart.space;
      }

      // increase lesson space 
      var lesson = this.getLessonById(lessonId);
      ++lesson.space;
    },
    // get lesson by id
    getLessonById(_id) {
      var lesson = this.lessons.find(u => u._id == _id);
      return lesson;
    },
    getCartItemFromCartByLessonId(lessonId) {
      var item = this.cart.find(u => u.lessonId == lessonId);
      return item;
    },
    showCheckout() {
      if (this.currentView === CheckoutComponent) { 
        this.title = 'Cart';
        this.currentView = LessonComponent }
      else { 
        this.title = 'Home';
        this.currentView = CheckoutComponent };

    }
  },
  computed: {
    cartItemCount: function () {
      if (this.cart.length > 0)
        return this.cart.reduce((total, item) => total + item.space, 0);
      return 0;
    },
  }
}
</script>
