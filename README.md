# point_map
Bazadagi Loacationni xaritaga chiqarish 

# models.py
class Store(models.Model):
    name = models.CharField(max_length=255)
    address = models.CharField(max_length=1000)
    phone = models.CharField(max_length=255, default="")
    location = LocationField(srid=4326, geography=True, null=True, zoom=7, default=Point(41.0, 69.0))
    region = models.ForeignKey(Region, blank=True, null=True, on_delete=models.DO_NOTHING)

    def __str__(self):
        return self.name
        
# url.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('store-point-map/', store_point, name='person_changelist'),
    ]

# views.py har bir regionda alohida turishi uchun
def store_point(request):
    region_id = request.GET.get('region')

    if region_id:
        region_ids = []
        reg = Region.objects.filter(parent__id=int(region_id))
        region_ids.append(int(region_id))
        for i in reg:
            region_ids.append(i.id)
        locations = [
            [l.name, l.location[0], l.location[1], i, l.address, l.phone]
            for i, l in enumerate(Store.objects.filter(region__id__in=region_ids))
        ]
    else:
        region_id = 21
        locations = [
            [l.name, l.location[0], l.location[1], i, l.address, l.phone]
            for i, l in enumerate(Store.objects.all())
        ]
    region = Region.objects.filter(Q(parent__id=21) | Q(id=21))
    context = {'locations': json.dumps(locations),
               'region': region,
               'region_id': int(region_id)}
    return render(request, 'map.html', context)
    
 # map.html
 <!DOCTYPE html>
<html>
  <head>
    <title>Stores</title>

    <style type="text/css">
      /* Set the size of the div element that contains the map */
      #map {
        height: 600px;
        /* The height is 400 pixels */
        width: 100%;
        /* The width is the width of the web page */
      }
    </style>
        <script src="https://unpkg.com/@google/markerclustererplus@4.0.1/dist/markerclustererplus.min.js"></script>


      <script type="text/javascript">
    function initMap() {
        var locations = {{ locations|safe }};

        var map = new google.maps.Map(document.getElementById('map'), {
          zoom: 6,
          center: new google.maps.LatLng(41, 69),
          mapTypeControl: true,
          mapTypeControlOptions: {style: google.maps.MapTypeControlStyle.DROPDOWN_MENU},
          navigationControl: true,
          mapTypeId: google.maps.MapTypeId.ROADMAP
        });

        var infowindow = new google.maps.InfoWindow();
        var markers = []
        var marker, i;
        for (i = 0; i < locations.length; i++) {
          marker = new google.maps.Marker({
            position: new google.maps.LatLng(locations[i][2], locations[i][1]),
            map: map
          });
          markers.push(marker)
          google.maps.event.addListener(marker, 'click', (function(marker, i) {
            return function() {
              infowindow.setContent(locations[i][0] +'\n'+ locations[i][5] +'\n'+ locations[i][4]);
              infowindow.open(map, marker);
            }
          })(marker, i));

        }
        var markerCluster = new MarkerClusterer(map, markers, {imagePath:"https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m",})
        }
</script>

  </head>
  <body>
    <h3>Dorixonalar</h3>
    <!--The div element for the map -->
    <form action="/store-point-map/">
  <label for="region">Choose a region:</label>
  <select id="region" name="region">
      {% for i in region %}
      {% if i.id == region_id %}
        <option value="{{i.id}}" selected>{{i.name_uz}}</option>
      {% else %}
         <option value="{{i.id}}">{{i.name_uz}}</option>
      {% endif %}
      {% endfor %}
  </select>
  <input type="submit">
</form>
    <div id="map"></div>


           <script
      src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBR2VPav7rT71KDVjGGAwr72bKuRb26Py4&callback=initMap&libraries=&v=weekly"
      async
    ></script>

    <script async src="https://code.jquery.com/jquery-3.2.1.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4=" crossorigin="anonymous"></script>

<script src="https://www.gstatic.com/firebasejs/4.3.1/firebase.js"></script>
    <!-- Async script executes immediately and must be after any DOM elements used in callback. -->

  </body>
</html>
