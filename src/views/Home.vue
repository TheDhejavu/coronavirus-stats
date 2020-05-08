<template>
  <div class="home">
    <h1 class='top-heading text-center'>Coronovirus Statistics</h1>
    <div class='container animate-me' data-transition='fadeInUp'>
      <h2 class='heading'>Summary</h2>
      <div class='flex flex--wrap'>
        <div class='col-s-33'>
          <div class='card '>
            <p class='text'> Cases: </p>
            <h1 class='data-count'>{{ numberWithCommas(data.Global.TotalConfirmed) }}</h1>
          </div>
        </div>
        <div class='col-s-33'>
          <div class='card green'>
            <p class='text'> Recovered</p>
            <h1 class='data-count'>{{ numberWithCommas(data.Global.TotalRecovered) }}</h1>
          </div>
        </div>
        <div class='col-s-33'>
          <div class='card red'>
            <p class='text'> Deaths</p>
            <h1 class='data-count'>{{ numberWithCommas(data.Global.TotalDeaths) }}</h1>
          </div>
        </div>  
      </div>
    </div>
    <div class='container animate-me' data-transition='fadeInUp'>
      <h2 class='heading'>List by countries</h2>
      <div class='form-group'>
        <input 
          class='form-control' 
          v-model='searchValue'
          v-on:keyup="search"
          type='text' 
          placeholder="Search by country"
        />
      </div>
      <div class='flex country-wrap'>
        <table>
          <tr>
            <th>S/N</th>
            <th>Country</th>
            <th 
              class='clickable' 
              :class='(filter.type == "TotalConfirmed")? "active": ""' 
              @click='sort("TotalConfirmed")' >
              Total Cases <i class="uil uil-sorting"></i>
            </th>
            <th 
              class='clickable' 
              :class='(filter.type == "TotalDeaths")? "active": ""'  
              @click='sort("TotalDeaths")'>
              Total Deaths <i class="uil uil-sorting"></i>
            </th>
            <th 
              class='clickable' 
              :class='(filter.type == "TotalRecovered")? "active": ""' 
              @click='sort("TotalRecovered")'>Total Recovered <i class="uil uil-sorting"></i>
            </th>
          </tr>
          <tr v-for="(country,index) in data.Countries" v-bind:key="index">
            <td>{{index+1}}</td>
            <td>{{ country.Country }}</td>
            <td>{{ country.TotalConfirmed }}</td>
            <td class='red'>{{ country.TotalDeaths }}</td>
            <td class='green'>{{ country.TotalRecovered }}</td>
          </tr>
        </table>
      </div>
    </div>
    <footer class="footer">
      <div class="footer-panel">
        <ul class='lists'>
          <li> <a target='__blank' href='https://www.google.com/search?rlz=1C1GCEB_enNG850NG850&sxsrf=ALeKk0107d2hI49JSMedIOpuCHQElMLNww%3A1588940334403&ei=Lk61XoGGGNXrxgPAqrnYCg&q=coronavirus+&oq=coronavirus+&gs_lcp=CgZwc3ktYWIQAzIECCMQJzIECCMQJzIECCMQJzIFCAAQkQIyBQgAEJECMgIIADICCAAyAggAMgIIADICCAA6BAgAEEdQwQtYwQtg5hBoAHABeACAAdwBiAHcAZIBAzItMZgBAKABAaoBB2d3cy13aXo&sclient=psy-ab&ved=0ahUKEwiB1c2BoKTpAhXVtXEKHUBVDqsQ4dUDCAw&uact=5'><i class="uil uil-google"></i> Google</a> </li>
        </ul>
        <p class='text'>© 2020 · Made with <i class="uil uil-heart" ></i> by bukola</p>
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'Home',
  components: {},
  data() {
    return {
      searchValue: '',
      filter: {
        type:"",
        order:"",
      },
      data:{
          "Global": {
            "TotalConfirmed": "",
            "TotalDeaths": "",
            "TotalRecovered": ""
        },
        "Countries": [],
      },
    };
  },
  mounted() {
    fetch('https://api.covid19api.com/summary')
    .then((response)=> {
        if (response.status !== 200) {
          console.log('Looks like there was a problem. Status Code: ' +
          response.status);
          return;
        }
        response.json().then((data)=> {
          this.data = data;
          this.CountriesCopy = data.Countries;
          this.sortData("LOW_TO_HIGH", "Country", this.data.Countries);
        });
      }
    )
    .catch(function(err) {
      console.log('Fetch Error :-S', err);
    });
  },
  methods: {
    numberWithCommas(x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    search() {
      this.data.Countries = this.CountriesCopy.filter(item => {
        return item.Country
          .toLowerCase()
          .includes(this.searchValue.toLowerCase());
      });
    },
    sort(type) {
      if(this.filter.type == type) {
        this.filter.order = (this.filter.order == "HIGH_TO_LOW") ? "LOW_TO_HIGH" : "HIGH_TO_LOW";
      } else {
        this.filter.type = type;
        this.filter.order = "HIGH_TO_LOW";
      }
      this.data.Countries = this.sortData(this.filter.order, type, this.data.Countries);
      this.CountriesCopy = this.data.Countries;
    },
    sortData(order,type,data) {
      var SlicedData = data.slice(0);
      SlicedData.sort(function(a,b) {
        if( order == "HIGH_TO_LOW") {
          return  b[type] - a[type]
        } else {
          return  a[type] - b[type]
        }
      });
      this.data.Countries = SlicedData;

      return SlicedData;
    },
  }
}
</script>

<style scoped lang='scss'>
@import "../assets/scss/_animations";
.home {
  max-width: 1000px;
  margin:0 auto;
  .top-heading {
    padding:0 20px;
    margin:30px 0;
    font-weight:400;
    display:inline-block;
    border-left: 10px solid $primary-color;
  }
  .container {
    border-radius:5px;
    margin:40px 0;
    padding:20px;
    background: #fff;
    box-shadow: 0 5px 5px 0 rgba(0,0,0,0.06);
    .form-control {
      padding:10px;
      border-radius:5px;
      width:100%;
      border:1px solid rgb(231, 228, 228);
      color:#555;
      margin: 15px 0;
      outline: none;
      &:focus {
        border:1px solid $primary-color;
      }
    }
    &.fadeInUp {
      animation: fadeInUp .7s ease-out;
    }
  }
  .heading {
    font-weight:400;
  }
  .card {
    padding:100px;
    .data-count {
      font-size:3rem;
      padding:10px 0;
    }
    .text {
      font-size:1.2rem;
      text-transform: uppercase;
    }
    &.red {
      color:$red;
    }
    &.green {
      color: $green;
    }
  }
  .country-wrap {
    margin-bottom:2px;
  }
  table {
    border-collapse: collapse;
    width: 100%;
  }

  td, th {
    border: 1px solid #f1f1f1;
    text-align: left;
    padding: 8px;
    position: relative;
  }
  th {
    animation: fadeInUp .7s ease-out;
    transition: 0.5s;
    &.clickable {
      cursor: pointer;
      &.active {
        background: #ccc;
      }
      &:hover {
        background: #ccc;
      }
      i {
        position: absolute;
        right:0;
      }
    }
  }
  tr {
    font-weight:400;
     &.fadeInUp {
      animation: fadeInUp .7s ease-out;
    }
  }

  tr:nth-child(even) {
    background-color: #f1f1f1;
  }
}

.footer{
  width:100%;
  .footer-panel{
    max-width: 1100px;
    margin: 0 auto;
    padding:60px 20px;
    font-size:0.8rem;
    .text i {
      color:$red;
    }
  }
  .lists {
    li {
      display: inline-block;
      padding:10px 0;
      padding-right:20px;
      font-weight:500;
    }
  }
}
</style>
