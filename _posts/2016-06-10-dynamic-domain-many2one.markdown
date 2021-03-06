---
layout: post
title: Dynamic domain on many2one based on checkbox
date: 2016-06-11T0:23:11
categories: odoo
published: true
---
Hey guys.

This time i will share my exeperience creating dynamic domain on many2one field based on boolean fields. In this tutorial i created dynamic domain for the product sub category(many2one) and the product category(boolean). So product sub category will display the list of content based on multiple boolean field. Interesting right. So lets get started.

## 1. Creating the fields

Create the object for the many2one relation product sub category that have parent category. That contain two fields parent category and name.

{% highlight ruby %}
_name = "product.sub.category"
#=> selection field for parent category. 
parent_category = fields.Selection([('is_living_room','Living Room'),
							('is_kitchen_stuff','Kitchen & Dining'),
                            ('is_bed','Bedroom'),
                            ('is_home_office','Home Office'),
                            ('is_other','Other')], string='Category')
name = fields.Char('Sub Category Name')
{% endhighlight %}

Inherit product.template object and add some custom fields. 

{% highlight ruby %}
_inherit = "product.template"

is_living_room  = fields.Boolean(string='Living Room')
is_kitchen_stuff= fields.Boolean(string='Kitchen & Dining')
is_bed          = fields.Boolean(string='Bedroom')
is_home_office  = fields.Boolean(string='Home Office')
is_other        = fields.Boolean(string='Other')
#=> boolean fields for checkboxes.
sub_categ_ids_m2o = fields.Many2one ('product.sub.category', string='Select Sub category')
#=> dont forget to create the object for many2one relation in this case 'product.sub.category'.
{% endhighlight %}



## 2. Create the function 

first we create the conditional logic when checkbox or checkboxes (the boolean 'parent category' fiedls) checked. for setting the domain in many2one field we need list, so this conditional code is all about manipulating the list.

{% highlight ruby %}
#=> use api.onchange. The function need to be triggered when the boolean fields changed.
@api.onchange('is_living_room', 'is_bed', 'is_kitchen_stuff', 'is_home_office', 'is_other')
def onchange_categ(self):
  #=> create the list that will contain the selected category for setting the many2one field.
  selected_categ = []
  res={}
  
  #=> here the conditional to append or remove the parent category string    	 into the list for   use to set the many2one.
  
  if self.is_living_room :
  	selected_categ.append ('is_living_room')
  if self.is_living_room is False:
  	if 'is_living_room' in selected_categ :
  		selected_categ.remove ('is_living_room')
  if self.is_bed :
  	selected_categ.append ('is_bed') 
  if self.is_bed is False:
  	if 'is_bed' in selected_categ :
  		selected_categ.remove ('is_bed')
  if self.is_kitchen_stuff :
  	selected_categ.append ('is_kitchen_stuff') 
  if self.is_kitchen_stuff is False:
  	if 'is_kitchen_stuff' in selected_categ :
  		selected_categ.remove ('is_kitchen_stuff')
  if self.is_home_office :
  	selected_categ.append ('is_home_office') 
  if self.is_home_office is False:
  	if 'is_home_office' in selected_categ :
  		selected_categ.remove ('is_home_office')
  if self.is_other :
  	selected_categ.append ('is_other')
  if self.is_other is False:
  	if 'is_other' in selected_categ :
  		selected_categ.remove ('is_other')
{% endhighlight %}

Now we write the code for set the many2one domain based on list that have been initialized through the conditional before.

{% highlight ruby %}
  #=> now we set the many2one field domain with the list of selected ids.
  res.update({
  	'domain' : {
  		'sub_categ_ids_m2o':[('parent_category','=',list(set(selected_categ)))],

 	 }
  })        
 
 return res

{% endhighlight %}

This code will only display the content of many2one field that 'parent_category' value is in the list. So this is it please comment below if you have some question. Thanks 



