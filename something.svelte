<script>
  import data from './datafile'

  let ethnicity = [
      'Asian',
      'Black',
      'Mixed or Multiple',
      'Other ethnic group',
      'White',
    ],
    sex = ['Female', 'Male'],
    status = [
      'Married or in registered civil partnership',
      'Widowed',
      'Divorced',
      'Never married',
      'Separated',
    ],
    age = ['65-69', '70-74', '75-79', '80-84', '85-89', '90+'].reverse()
</script>

<style>
  .bucket {
    margin: 5px;
    padding: 5px;
    display: flex;
    flex-wrap: wrap;
  }
  .status {
    display: flex;
    flex-wrap: wrap;
    flex-direction: column;
    margin: 5px;
    padding: 5px;
    border: 0.5px solid grey;
  }
  .sex {
    display: flex;
    flex-wrap: nowrap;
  }
  .age {
    display: inline-block;
    width: 50px;
    margin: 2px;
  }

  .green {
    background-color: #6749a6;
    width: 100%;
    height: 100%;
    float: right;
  }
  .male {
  }

  .female {
    flex-direction: row-reverse;
  }

  .blue {
    background-color: #2ea1a4;
    width: 100%;
    height: 100%;
  }

  .filler {
    min-width: 5px;
    min-height: 5px;
  }
</style>


{#each ethnicity as ethnic, i}
  <div class="bucket">
    <!-- {ethnic}
      <br />-->
    {#each status as stat, ii}
      <div class="status">
        <!--{stat}<br>-->
        {#each age as age, iii}
          <div class="sex">
            <!--{sex}<br>-->
            {#each sex as sex, iiii}
              {#if sex == 'Female'}
                <div class="age female">
                  <div
                    class="green"
                    style="width:{data.find((e) => e.age == age && e.ethnicity == ethnic && e.sex == sex && e.status == stat)['Divorced'] * 120 || 0}%">
                  </div>
                </div>
              {/if}
              {#if sex == 'Male'}
                <div class="age male">
                  <div
                    class="blue"
                    style="width:{data.find((e) => e.age == age && e.ethnicity == ethnic && e.sex == sex && e.status == stat)['Divorced'] * 120 || 0}%">
                    <div class="filler" />
                  </div>
                </div>
              {/if}
            {/each}
          </div>
        {/each}
      </div>
    {/each}
  </div>
{/each}
