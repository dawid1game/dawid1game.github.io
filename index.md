<!DOCTYPE html> 
<html> 
  
<head> 
    <title> 
        Detect touch screen device  
        using JavaScript 
    </title> 
</head> 
  
<body> 
    <p> 
        Detect touch screen device  
        using JavaScript 
    </p> 
      
    <p> 
        If touch screen detected then display 
        an image otherwise image will not 
        display     
    </p> 
      
    <div id="GFG"></div> 
  
    <script type="text/javascript"> 
      
        function is_touch_enabled() { 
            return ( 'ontouchstart' in window ) ||  
                   ( navigator.maxTouchPoints > 0 ) || 
                   ( navigator.msMaxTouchPoints > 0 ); 
        } 
      
        var src =  
"https://contribute.geeksforgeeks.org/wp-content/uploads/gfg-39.png";  
      
        if( is_touch_enabled() ) { 
            var img = "<img src=" + src + " height='100'/>";; 
        } 
        else { 
            var img = ""; 
        } 
      
        document.getElementById('GFG').innerHTML = img; 
    </script> 
</body> 
  
</html> 
