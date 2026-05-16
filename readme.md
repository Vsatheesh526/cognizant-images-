<div class="row-cards">

<div data-sly-list.parent="${resource.listChildren}">
    <div data-sly-test="${parent.name == 'items'}">

        <div class="welcome-container">

            <div data-sly-list.item="${parent.listChildren}">
                
                <div class="welcome-card">

                   <img src="${item.valueMap.icon @ default='/content/dam/default-icon.png'}"
     alt="${item.valueMap.title @ default='Card'}" />

<h3>${item.valueMap.title @ default='Default Title'}</h3>

<p>${item.valueMap.description @ default='Default description goes here.'}</p>

                </div>

            </div>

        </div>

    </div>
</div>

</div>



<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"/>

<sly data-sly-call="${clientlib.css @ categories='corporate-hr-portal.site'}"/>
<sly data-sly-call="${clientlib.js @ categories='corporate-hr-portal.site'}"/>
