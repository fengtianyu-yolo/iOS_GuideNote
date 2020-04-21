# UIKit

<style type="text/css">
	.container {
		display: flex;
		display: -webkit-flex; /* Safari */
		flex-wrap: wrap;
		background-color: rgba(245,255,250,0.5);
		/*justify-content: space-around;*/
		justify-content: center;
	}

	.item {
		background-color: #FFFFFF;
		box-shadow: 5px 5px 10px rgba(220,220,220,0.7);
		height: 140px;
		width: 230px;
		margin: 30px 60px;
		text-align: center;
		line-height: 150px;
		font-weight: 700;
		font-size: 1.3em;
		border-radius: 5px;
	}

	.item:hover {
		background-color: rgb(240,248,250);;
		box-shadow: 10px 10px 20px rgba(220,220,220,0.7);
	}

	a {
		color: blue;
		text-decoration: none;
	}

</style>

<div class="container">
  
  <div class="item">
	  <a href="#">UIButton</a>
  </div>

  <div class="item">
	  <a href="#">UIView</a>
  </div>

</div>
