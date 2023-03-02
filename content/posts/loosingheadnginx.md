---
title: "Loosing head over nginx"
date: 2022-07-08T12:12:20+02:00
lastmod: ":git"
draft: false
tags: ["python", "flask", "nginx", "gunicorn", "jinja2"]
---
# Loosing Header over nginx

In my app I serve a booking form - `newbooking.html` - , after sending it I want to present the user a `detail.html` page, showing the content of the just sent booking in a table and under the table buttons. The first button should always be visible, it's 
there to send the admins a message concerning the booking (users can visit the detail page for all their bookings, not just a recently sent one). The second one should only be there if the user got redirected to the `detail.html` 
page from the `newbooking.html` page; so after submitting the `newbooking` form. This button simply links to the `newbooking.html` page and says something like "send another booking"

## Initial solution - `request.referrer`
My initial solution was to check for the `request.referrer` in the `jinja2` template and if it matches my domain and the path for `newbooking.html`, show the second button.
So in my routes I check if I have a `validate_on_submit()` do stuff to the form data and the `redirect` to the `detail.html` page passing along the dynamic uuid.
**routes.py**
```python
@bp.route('/newbooking', methods=['GET', 'POST'])
def newbooking():
    
    form=BookingForm()

    if form.validate_on_submit():
        uuid = generate('123456789ABCDEFGHJKMNPQRSTUVWXYZabcdefghjkmnpqrstuvwxyz', 8)
        #do stuff with the data from the form

        
        flash('You just sent us the following booking:'
      return redirect(url_for('detail', buuid=uuid))
```

**jinja2 template**
```html
{% extends 'base.html' %}

{% block app_content %}
<h1>Details for Booking: {{ buuid.booking_uuid }}</h1>
<div class="row">
    <div class="table-responsive">
        <table class="table table-condensed table-hover">          
            <tr>
                <td>BOOKING CONTENT HERE</td>
            </tr>   
        </table>

        {% if request.referrer != None and request.referrer[-11:] == "/newbooking" %}
            <tr>
                <td colspan="2"><a href="{{ url_for('bookingmsg', buuid=buuid.booking_uuid) }}"><button type="button" class="btn btn-warning" style="width:100%; margin-bottom: 15px;">send us a message for this booking</button></a></td>
            </tr>
            <tr>
                <td colspan="2"><a href="{{ url_for('newbooking') }}"><button type="button" class="btn btn-success" style="width:100%">make a new booking</button></a></td>
            </tr>
            
            {% else %}
            <tr>
                <td colspan="2"><a href="{{ url_for('bookingmsg', buuid=buuid.booking_uuid) }}"><button type="button" class="btn btn-warning " style="width:100%; margin-bottom: 15px;">send us a message for this booking</button></a></td>
            </tr>
            {% endif %}

    </div>

</div>

    
{% endblock app_content %}
```

This worked fine on my local setup. The client receives the `request.referrer` (one can check it in the developer console with `document.referrer`).
So I happily pushed to production.
It stopped working.

## Production server problems
My production server is a small AWS Lightsail instance running debian and service my flask app via gunicorn and a nginx reverse proxy.
```nginx
server {

listen 443 ssl;

server_name app.info


access_log /var/log/access.log;

error_log /var/log/error.log;


location / {

proxy_pass http://localhost:8000;

proxy_redirect off;

proxy_set_header Host $host;

proxy_set_header X-Real-IP $remote_addr;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

}

ssl_certificate path/to/cert

ssl_certificate_key /path/to/certkey

}
```
Surfing on my app the referrer normally showed up fine: `app.info/whateverpage`.
BUT after submitting a form and getting redirected to the `detail.html` page left the referrer empty. So obviusly my `jinja` template never showed the `make a new booking` button.
My guess was that there is a problem between the nginx reverse proxy and gunicorn, which removes the referrer, when landing on a page after a flask `redirect`, since it worked fine after flask `render_template`.
I tried building my own responses and including the referrer in the header - to no avail.
I was ready to give up. I already spent several hours making this button appear in production.
Then I found a workaround!

## request.args to the rescue
I appened a parameter/argument to the flask `redirect` after submitting the form!
All this does it append `?redirected=True` to the url. So hitting `detail.html` after the redirect shows up as `https://app.info/detail/<buuid>?redirected=True`
So my updated `routes.py`:
```python
@bp.route('/newbooking', methods=['GET', 'POST'])
def newbooking():
    
    form=BookingForm()

    if form.validate_on_submit():
        uuid = generate('123456789ABCDEFGHJKMNPQRSTUVWXYZabcdefghjkmnpqrstuvwxyz', 8)
        #do stuff with the data from the form

        
        flash('You just sent us the following booking:'
      return redirect(url_for('detail', buuid=uuid, **redirected=True**)) #added parameter here!
```

updated part of the `jinja2` template for `detail.html`:
```html
{% if **request.args.get('redirected') == "True"** %} 
     <tr>
                <td colspan="2"><a href="{{ url_for('bookingmsg', buuid=buuid.booking_uuid) }}"><button type="button" class="btn btn-warning" style="width:100%; margin-bottom: 15px;">send us a message for this booking</button></a></td>
            </tr>
            <tr>
                <td colspan="2"><a href="{{ url_for('newbooking') }}"><button type="button" class="btn btn-success" style="width:100%">make a new booking</button></a></td>
            </tr>
            
            {% else %}
            <tr>
                <td colspan="2"><a href="{{ url_for('bookingmsg', buuid=buuid.booking_uuid) }}"><button type="button" class="btn btn-warning " style="width:100%; margin-bottom: 15px;">send us a message for this booking</button></a></td>
            </tr>
            {% endif %}       
```

The only caveat is that users can manually append `?redirected=True` to their url. However, since the only functionality is to show or not show a button linking to a page they find in any way also in their navbar, I don't see any problem (please correct me if I'm wrong)
