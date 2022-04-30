<template>
    <!-- MENUBAR -->
    <div class="place-content-center flex my-2">
        <table>
            <tr>
                <td><label for="desde">Desde: </label></td>
                <td>
                    <input v-model="desde" type="date" id="desde" class="mt-1 block w-full py-2 px-3 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm w-48 mr-8">
                </td>
                <td><label for="hasta">Hasta: </label></td>
                <td>
                    <input v-model="hasta" type="date" id="hasta" class="mt-1 block w-full py-2 px-3 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm w-48 mr-8">
                </td>
                <td>
                    <button @click="Send()" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded mx-2">Buscar</button>
                </td>
            </tr>
            <tr v-if="existeError">
                <td colspan="5" class="my-8">
                    <span class="bg-red-300 rounded mx-32 px-8 text-white">
                        {{this.mensajeError}}
                    </span>
                </td>
            </tr>
        </table>
    </div>
    <!-- contador de asistencias -->
    <div v-if="this.desde == this.hasta" class="place-content-center flex">
        <span class="text-xl">
            Asistencia: {{this.arrayData.length}}
        </span>
    </div>
    <!-- TABLA PRINT -->
    <div>
        <div class="place-content-center flex my-2" >
        <div class="my-2 sm:-mx-6 lg:-mx-8 overflow-y-scroll h-96">
            <div class="py-2 align-middle inline-block min-w-full sm:px-6 lg:px-8">
              <div id="table-pdf" class="shadow border-b border-gray-200 sm:rounded-lg">
                  <table class="min-w-full divide-y divide-gray-200 ">
                    <thead class="bg-gray-50">
                        <tr>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Nombre
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Departamento
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Dia semana
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Fecha
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Hora entrada
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Hora salida
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Min Trabajados
                        </th>
                        <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                            Permiso de asistencia
                        </th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        <tr  v-for="item in this.arrayData" :key="item">
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="flex items-center">
                            <div class="flex-shrink-0 h-10 w-10">
                                <img class="h-10 w-10 rounded-full" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAOEAAADhCAMAAAAJbSJIAAAA21BMVEUUfqv////m5ubl5eXn5+f4+Pju7u77+/vx8fH19fXr6+sAfKzz8/Pf398AfawAd6cAa5gAcJ/X19fLysoAaJHZ1NEAZo3s6OWEkJgAeavh3dq+vbwAdaMAcJ2cpasAcaLHz9Ta4ue1zt5XfJEzaoa/wcNRepFkg5YAYoevsbKOmaCip6sAaJbIxMIvb49jeYc/cIp3jJmuw9C0ur6OoKyBmKedqrJBc41vkqYnZIJwhJBpiJtIb4RaeYp4ho+jpKaGobFag5uFjJFqd3+anJ4ecJVLf5s6Zn6Up7JL4aE5AAAVHklEQVR4nOVdaXviONbFmywv2CzGBkOSeV8SEpOGhKlKaJJKpbsn6Zr//4tG8r7IqwTkmblf3HWfNOYg6d6joyupx3GcxIu8hJ6CyAP0ALwopLx8ey9MvAp6ig28cuwVyV61pVePvL3/DYQCL/gI+fBb80JHL8+H37rCKyOvUuoVeV5l6+1JkqQAABT0RA+IHjp6/hd5ewiq3xhCrokYeaEQ/8BBcwpq1qu09IptvT2ROKA6DEl4hCHZdvCRvP8DbZjtwTqxXzPwyrKs6DpEDwlCXUJPCNmOOCn0wvw4jH52WVf6/T6v6AqPnujbAPQAeoVXQP8UE68aexXs1UOvosiq0HedyXjuzf/5/8j++c2bjyeu6/aBLGX+Fj3xJ+jB55K9sJEXynocSzNDx89FAA2Lohc28YoZL6cCRXHG377fXm8ta7AwE7uwrMP18unbjetAoIZDp/gJuSypF7xq0av6Xj2XD4Vg6AjhgBIEWq8ocpwzfn7a26Y504we0QwDYbWuf/9+4+IvFgw+IRxQQtACQmuv6Ht1PfL2GMIKvTxGN99dW6XQcji12eDw8s3lJFGkg0X0Ms8WHHTnT4fBzGiCLoVzdnX36k2iDkudLYR0tqClaokXDQfo3dpmS3QxStN6vHTxF6OmaqlP6IUhF9IHbV1xvZVldkMXgdQGjx8TFI5pCZwYe7PMu3uaF1Vp/Ls9o4IXmmb9OXeASJf8UaQhZIvusycZuB/bK40BPN8M83M6BlRDMp8taNtwsrOYwQtMu3gbM2rDUqLVdAzI0vhhwBifj3H4ONc5BuOQOpaOV8Mj4MNmmNeeTB9LqfKhyE1WQxbRpQzj7DCXVMp8SMNewOT2mPh8Mx/n0KedXZkOBS/VufXg2PiQGVdvDuBbUjUOJry0c7bQ71nHzzLTfps6astsAam1NtVdmqfB52PcbpT68JL26nqF1taAwMnc5Sk6aGLG1Q7KbRKHSqe1gdH+hA0YmHb3TT2Z1iafuAFDu3rpOntq24bO68kbMDBtPzqW1pbu7dLYPlEILZoxuJfaj8OWsRSFmHPhw2Y+SK1jact8+HCmHhqZ9svplA+bchp3e7YeGplhzbkGohRIOE2bSDO6O0cMzdvAI8QUgRhp2mpt87MOwcTM9ZG0tsvhuaFFZr5wOiOtLeXl1lfnBpaY9kOH1QSOb6+1fSWACOJKqk7+oLXWNj1zlsib9kOqzBagrdb21QD6ENWqNoy95VpbyvsFAWKISvk4hIIeLZY2iqXrLwgQQVzK5bG030pru/xSQSYx7a08H/b1OB/WUzXviwLs9WY/ywgc34cxp6mNNOMvwmRIZt6XRRpVbDy3cO1zw6gyzFHJKzNitKBcNz+UPr8C2S633yYl80OxodbGvX1tgL3eHVSIc3y1mdYmfc08kTZtJXMEApeKNJXZYvxlphPlNlsTswXkmmhtzpeOMpEN5qVtWKe1SSuGg9AwDE0zOlZpVJvtwBQKGI5DWK+1AXaD0DDt1e3t7vJpubXNRnVErT596RRiKWiQD2VmqV67Wnr4O2CTnPn00WKM0rws5EMoiLVam7xl8y2MIa45yBiuCWO69D9wC3VtQq3WBqZsvsJsu+EIJk+m2wt25Sl7wOciDYgjTUm2UNkkCmPwTsIX2HiKBiWLt+B+CtLZAs2eAFejtYG/mZQ3bSflAPHrNq+MFpItN7O1IURYobUpTOLo7KESXzAm10xWeoyllFEqBLVaa+PhbwzeOvxeDxBj3F0w6C/mXM3oNP3qbAFfGPyuQ68RQGTuI4PXbZ2M1pbLFrk2ZBJmzIoYkzd5R//C2RqQ25CktcmP9N1mtmsOENkHPUTLSWttYqXWdkMfZoy3VgA5zqOGqD0l+VDsq1yV1nZN34S22xIh99eM9p2WqySzJx9hidbGefRNuGgcZRJ7oA032oOUaG0+whKtTaEnpK37KMfkvZieRnOLqmzBoAkHea7dyOa0LzZwdMtlC5LWRh9IOzUhsl+0b7bcuA1Rc5VobfLNghZgzyROJ+qNuvdoUy5EoQrRjrbC7Aks6QPpQe6GUNrSvvnTLUaaXLYQNxfUAI12yT5lO9pwar6H+4pS2SLXhmBH34SLFnwta9Td1Lh20m2YaG0wGIfo4VjUAHsXHYchouDUb1/MFX9vqspDH1RBa9M9amaBEFZPe6uMmk0ZrzCMpeR8KCp7eoA9G3RGSB/mLDfIhzxZaxMnLNQZW++MkDrU9Mx71a/VT2tt6bkFE4HNhp0RTunj3D6v02SzBZOFCgqEa/pfGJFTjDC9MpNqwwmTJXvbqQNyTITac4KwoLVxT0xk7vMiRN0UMTcByCStDVCzJt8oELKIA4MJyKxyc6nzAhgtiFLkw1cGCLUP6B9ZQNDa1Es2+nN3TsOxWLE0lkqmDSE+iEOSdERyuEf6j8fWRcIIjM16l+VAKOCpE8KW0dp4nYXQjUy77IqQnpdim83VdCxN8qE6ZrTm2332NKefffewrKim82GitanfGS11GT+6ImS0ZrmHJXVtjIZhr7ftipCBvoDNguS6Np1ZbYnVcXIhs8nHWCdSSXVtG2alF1bHlM9i+o0NhTpVLda1cYyyIa5P7gaQUxj1ImPPiaCotYHfWVWAbDuzNha0FJsN+4RdQfDA5tMp0iGzbjqY9It7SHlWn95bdCdtrELN7Ca1h1RRJAWRG0WasAo0i06LFoExWNfDpj2rMNTaYBJLGSyLBmbedAYIGYUa7Q8FFPPhv5mF0ufOCJmVe+5lUNTa2MzvkRnTzgjH9EsKgR1kEGttUSyFzIpJu9NSZtkCL3hz+bmFwyiM9WhUjFtWv7K5CY4o5BOtDU5YJQu8dtARoMwqJffMd5lLa22yJOsMEWpd54dMFPfgK3zIkiRj5iZH2UIeMdzd9Kl0Q8hoeohMWyv5PaTyiOHOitlf3ZqQXTcyXpT8HlL5/1juHRl00RNZ1JpFZiyTNvS1NsTc/sESYaeRuGG4fcV4xIeUZLQ2yBRh77EDwg+Wte37JJZGJ02zRWh1WH5iuols6yNMa22MEQ7bD0RWrDuwTx9hrLXhWMo00qBo3RrhholWGtlnFEuFIyHswE2ZcVLfcgiZZ4uwTLCNKXdM348QJlqbrKhAlRgjbN1NGVREpu1TAQBvSJDUSKdhHGl6vTupHULqusSspSLNcbJFr3fVboLBjnQH9plki2O1obFshfCF8abEfbw7uKcgw1ob43HYkptOWB9r8Igpm4/tWLG0ZSMyJN2B7ZW81sZ09hRYi6n+nPXRG8atkj+v7QgIm4dThZl6EZnxU8prbXDMlDT5Nmu6gMFubh+Z5m+bC5SoGCG7CXZsVrNgM2d/roH2kSCMtbZP5q9Bc5gmdZhsCjCyZvoFLxmtDQLmY6HnnwJUa6z2jGdssZHknNYmcX8e47gDs56estjMWTDLAcXz2p6OcqzlsC5lrNlHOGQHCRTr2p6Pc3Dn8L4S4F9HOR7G+FMCGa0NkRtdZ7Z+mLOLiio3+ek4L9X+UPWAjUIlrmsTJ6xWtvJ2VdqK+huDvQ8k0zzC3Qgis3X8gi2m5H1Q46OdaBut42fr2iCjNfS0fQYYZj8IqV8O7x04BsqLSarahIurvpjV08R2N1KWfj80BtPcmqLshQ043B2hJe84FUR3laUQMquJCs3wSdtfwbRBs9cpbcq5vw5uTcLnHnM/mc8slghhXmtDCJmVm4SvuQu6phceUa8NVuuN67qT+XplhSfwmCu/ae8ttt1Hu8cIo2yBLyZUgarIss5UzzN/RW02OYSdw9BMC1l8HZsxWIchyH1jGsiHYw4f16ZKspxobUAQ9B/sfknNTu1BlNakA4UMMx1/5ixHo+2IpLo2UWXGarTBSzawTB7y5wlpi+ssE1DW7K482QORUNcm8CKjOm/Nui0mB3d9WMS90zDtl2LxmzM90F3aFn+B7yrfT9e1BVqbokiARc437R153qtvpqtP27bsz+V6Ti4iVrwl5c10wVfYSJII81obllToi4YM83BfVQEtQcepXll03n9Q91Zcz6MS6tqQ957qsw3TuvU6VmGkzf2+pWpI4xZyfsYv7iHlaXRZbfh4331HV9aU+SPFTWfaO8AIU1pb6gLZrtRUG27XFFWlBJvs7rpepjhw8H78+FrZ7B7STquUxtV22n23WqnBy+tOV54Zq+S0a7GwS9Ztr+sZw2X3qucam7z+1v4nR5Qte543Ijf4LmmImBt6tC5QNPdHaL7E3JfW7Wi5iKoBAd+TDVQ1rbXhEwfUlt3UsDpvxGtqbafJxgr4p10TtDb/xIF26qx23fo0qPamtLu8x/RUUS1mi/gUJaWNaupfMnECa3XCIWLduTaMtLaQubUoGDD+Pg1Aro2Ua+xkn4YKodamK/nz2loUQw/ZZsAKg83HzgVWoCvvRmh+MKvW4GBLVnbftGcZP6ITeErvIeUbl7F23YLXyZqSLdMTueAEHpLWxvE+gWtYFaE9nRAg13D91jjIAVUja23heW0Njx04aRM2bUS86uxTtYLWJgqpE+kabcY95SjE9t5I/7+D0XltKYS+1qZirU1GD0Tgmi06ny6QBtao+lSbciFVkxEJVfNaW3y6p7xv0Iidt2t3tSbbsmxXjE+7rrpZTm5QUk6xT7SjNdCrtamUnO5ZfQ9pvV5jnYCQ5uxQ+6VsGF9FRtbalIjAKbXhFM0yT261VERbSzGKEq0tvhsB1oVTrXrp+ihWW59pO3xyNwJZa4tPu1brJKnWNc4MrG52bnogddo1WWtLCByomQl32TBCbdVnuRl7kL4boUxri5uzekOu1n0fLIV5lczNT9AwomqlWlsiDlfOE82uWyiprFJ/0F5wgVnqboQSrS0hcPC2ok+0P8qaiVUVu386udsfkvlhQNVkiJ5pAgfc8mBjnGUYVuYL8xtEVE3WI6oGBIWstaWTf/ms8zzDsOqsaO0lfzVwmdaW7rDlzOY8w7BqIOKpXPYmnVKtLdWG8qjsA4fnGYaIuJV8oSus2uZuByzT2tIETi6rL6c425LOSnYoajsuj0IXYJnWliFw5BxLcWYCpZF5iLGV4ntIk7sRmt1DqhPLd88VaMqOc8NbVzrfQ0rkp+bR1yrKjJjB/K8j1txDirAmWltM4JCXeMsqxdGWlKYQyLf2IsVUTYypGqeq5VpbtsNOiwoQxfGktFasaULsQ4+JDB9TNdxM5VpblsD9yI9u4/psAAmh785JUbXU3XlQLNfacgTOyUebltvumFqhWGQw5nyqJmOqFqAItDYstZVqbdnkLzq5gj6KI4SoLc/bBvE1VkLHe0h97ygbwbSzhdKCkmFecqW3Vje7hzT03mQgnouVYsvWopvTHFUj30NK0NryBE7y0hBPrXanLbOl3fyp6LnvGy+LoiCiV2lteQKnpNPi2Xg3tlRCnL0CMXetY8t7SJM1N+RNQTxjOkwnxNkuzO3ke7nzN8uR7iHNep/jMX7NoDivs8WLm7OX4pXxaa0NCGVaW0jgCl4QjcXzzSywRbML8ydQfVImpqkavlE99Pp3I1RpbcUOy3nBWdjdD3pmYaEEaE4lkUDViveQVmhtBO/Y3zNg/Otc6LC9+wgHlykymgovaYTJ3XklWhvBy00+McTFz/MNxHc/Hw68EqqWeFGgqdfaCAROv8bDQDucKV8oD3iiY9hjUELV0pGm5h7SQrYIvdKt6b+j84VONLbxVxBxLZ1QQtXSWlv1PaTlBI5b+yF18a9TFXwl9uG/2XyQy6ka+R7Scq2N5NXlse3/kocTT/SdtzDGoO9UStXSWpvYSGsjep1Hf9o/mJ4y4Hh3fgSwbwqqGp+JpU3vIa0Zkpf+bjTN/nYqfJNHX2Qz8V6jqsGXnj1V3UNaR+A4af6Jf1Hj6k04BT4p2GyqDS7xTrcqqpbW2rL3kJZRtVKvshsGL21/LFtrm3/6o8Jcuf7lMVyGqvkHXhS8YSwNvPXZguwNmrE323c/2rqRjV79WY0xwAU8kKSqkb35e0grtLYSr+pMg72h5vUR5/zu25X/Q5rLSXaJl0zVUl41QVintZV6lfHfwc9rHqsdR6/Bpeua7ekgQ8rib0bwBgROFCJvE52mbPkNhl3VMB+PsKnEfQg6iTFYh/dS+k3EV1C1tE5Teg9pYwKHvapzaYcY98y2dfkmectg16kx2E1AKNrXUjVytujehtjLKdNw/+vs7oUZzXHW20XQP4dvriQSSBmZqpEzfgOqVuXVJfcpaEf0dZg0pDRfhXsstcHb2C9VI3yH8J+lXoFvpbVVeNGcCv3kdiQu2C9zKpDKZnq9CPfvX6BOIZJJWb1XLbmHtMngI3n1+324A8tYWKv7jvNHx3uxo+3Q5ue0UH1QO/gqZk8NqFqlVxad+a0VLsQZM2t/OW7ZlGDivdnR5ljU3z3X3zgRkjIyVasicGqfL2ptmOQUqRrBC4leyb1/jLbSGdrQ+rXz3EbTD9mdr39Ywxjewl6PMQNVC2+rpGrYq0de1Y807bS22g4LRaBOvm+H0a77nmZad7drb+M6Zdd0Q3fsXb5sLTP+f4yF/Tp3gFhHymq9YbZop7WVeWPc/tCerB8HyQ5eQzMXC/uwmn68e/P5xHUdx3Xdzdx7f58uf90NzQQcbvjDDt/awNeTsgJCJlpbM68EFdd7tYeZfcqGppm+DSzLGl7h/9K0zB/MTGt1OQaQqyNlzbygr0betjpN7BUqvAoAY293GM60BtvEDG12dXf7vHFAZvqXyQANqVo60rDKFrDMiwe8s3n+Y29Z5szQCHU5qF011Kb73//tuZI/ZsJYn/pcdtmiug35Rl6Fy7MDFNLQu5zJt/XD8nprWQszsgvr87B6mD5PXAn1MzH1CXwJKQuHWWOvnmhteqhdqaMRQt8fjUZ4dowe/fCfvKgKgVcNvCr2ioG3L/L+36qhly/+bb+PxikQ3Amy0ebmZjzC/zURIAB8vz8S1PBtfPByXsCPPurvI/xEfzEej9F/+A/0aegx7o+qvP7/MlJ9TpnR2jLSBR9nnWiFJ5sPoxKdtl4clbhkjSj7t3r2bYUcl/Wq5V4OPZrVtbEZkpE3zFu13u6Dj+jtorV1oXWJtzH9ak/ViF6y1taYquUoFb23CVWrJXBqunO31toEYocViF2z1Evomu1IWb03XmDsqrVVEjiY8/J5r1LwNtHPuhG47lpbhqrVeUFLbyeqRva202mqqBrZm0rdmUjTgJQxInD/AYdHFK75tTjBAAAAAElFTkSuQmCC" alt="">
                            </div>
                            <div class="ml-4">
                                <div class="text-sm font-medium text-gray-900">
                                {{ item.NAME_EMPLEADO_CORRECT }}
                                </div>
                            </div>
                            </div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="text-sm text-gray-900">
                                {{ item.NAME_AREA}}
                            </div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="text-sm text-gray-900 text-center">
                                {{ item.DAY_ES}}
                            </div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-blue-100 text-white-800">
                            {{ item.AUTH_DATE }}
                            </span>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            {{ item.HOUR_IN.substring(11,16) }}
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            {{ this.isNull(item.HOUR_OUT) }}
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            <span v-if="this.ItemColor(item.HOUR_IN, item.HOUR_OUT, this.isSabado(item.MIN_WORK_DAY, item.DAY_ES, item.NAME_AREA)) == true" class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">
                                {{ this.CalculateHourWork(item.HOUR_IN, item.HOUR_OUT) }}min <span class="font-bold">/ {{  this.isSabado(item.MIN_WORK_DAY, item.DAY_ES, item.NAME_AREA) }} min</span>
                            </span>
                            <span v-else-if="this.ItemColor(item.HOUR_IN, item.HOUR_OUT, this.isSabado(item.MIN_WORK_DAY, item.DAY_ES, item.NAME_AREA)) == false" class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">
                                {{ this.CalculateHourWork(item.HOUR_IN, item.HOUR_OUT) }}min <span class="font-bold">/ {{ this.isSabado(item.MIN_WORK_DAY, item.DAY_ES, item.NAME_AREA) }} min</span>
                            </span>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            {{ item.PERMISO }}
                        </td>
                        </tr>

                        <!-- More people... -->
                    </tbody>
                  </table>
              </div>
            </div>
        </div>
        </div>
        <div v-if="hayDatos" class="place-content-center flex my-4">
            <button @click="exportPDF()" class="mx-4 bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded my-4">Imprimir PDF</button>
            <button @click="exportPDF2()" class="mx-4 bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded my-4">Imprimir PDF 2.0</button>
        </div>
    </div>

</template>
<script>
import {getURL} from '../config.js'

import jsPDF from 'jspdf'   
import 'jspdf-autotable'
import html2pdf from 'html2pdf.js'

import axios from "axios";
import moment from "moment";

export default {
    name: 'HomeView', 
    data(){
        return {
            permiso: sessionStorage.getItem('permiso'),
            id_area: sessionStorage.getItem('idarea'),
            desde: '',
            hasta: '',
            mensajeError: '',
            llegoMeta: null,
            existeError: false,
            data: null,
            arrayUser: [],
            arrayData: [],
            hayDatos: false,
            arrayPDF: [],
            arrayDateUser: [], //datos ya por usuario y ordenado de fecha
            dataEmpleados: null,
            arrayNameAllEmpleados: [], //props a recibir en MarcacionFaltantes!
            arrayNameEmpledosAsistence: [],
            arrayProp: [],
            faltantes: false
        }
    },
    methods: {
        LoadDates: function () {
            var fecha = new Date(); //Fecha actual
            var mes = fecha.getMonth()+1; //obteniendo mes
            var dia = fecha.getDate(); //obteniendo dia
            var ano = fecha.getFullYear(); //obteniendo año
            if(dia<10)
                dia='0'+dia; //agrega cero si el menor de 10
            if(mes<10)
                mes='0'+mes //agrega cero si el menor de 10
            this.desde = ano+"-"+mes+"-"+dia
            this.hasta = ano+"-"+mes+"-"+dia
        },
        backDataEmpleados: function () {
            let URL = getURL();
                axios.get(URL+"Empleados/AllData")
                .then((response) => {
                    this.dataEmpleados = this.BugMultiplicateUser2(response.data);
                    var dataClean = []
                    this.dataEmpleados.forEach(item => {
                        if (item.NAME_AREA == "Administracion" ||
                            item.NAME_AREA == "ATM" ||
                            item.NAME_AREA == "BI" ||
                            item.NAME_AREA == "Cumplimiento" ||
                            item.NAME_AREA == "Diligencias" ||
                            item.NAME_AREA == "IT" ||
                            item.NAME_AREA == "KYC" ||
                            item.NAME_AREA == "Legal" ||
                            item.NAME_AREA == "Operaciones") {
                                
                            dataClean.push(item)
                        }
                    });
                    

                    this.GroupByName(dataClean, this.arrayNameAllEmpleados)
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        Send: function() {
            this.clean()
            this.backDataEmpleados()
            var fecha = new Date(); //Fecha actual
            var dia = fecha.getDate(); //obteniendo dia
            if(dia<10)
                dia='0'+dia;

                if (this.desde != '' && this.hasta != '') {
                    this.clean()
                    this.existeError = false
                    let URL = getURL();
                    
                    if(this.permiso == 3){
                        axios.get(URL+"AsistenciasPorFechasSegunUsuario/"+this.desde+"/"+this.hasta+"/"+this.id_area)
                        .then((response) => {
                            this.data = response.data;
                            //this.GroupByUser(this.data);
                        })
                        .catch((error) => {
                            console.log(error);
                        });
                    }else{
                        axios.get(URL+"AsistenciasPorFechas/"+this.desde+"/"+this.hasta)
                        .then((response) => {
                            this.data = response.data;
                            this.BugMultiplicateUser()
                            this.GroupByName(this.arrayData, this.arrayNameEmpledosAsistence)
                            this.hayDatos = true
                            this.DatosForPDF(this.arrayData)
                            this.FilterEmpleadoNotAsistence()
                        })
                        .catch((error) => {
                            console.log(error);
                        });
                    }


                }else{
                    this.existeError = true

                    if (this.desde == '' && this.desde == '') {
                        this.mensajeError = 'Ingrese los rangos de busqueda!';
                    }
                    else if(this.desde == '') {                   
                        this.mensajeError = 'Ingrese fecha desde donde desea buscar!';
                    }
                    else if(this.hasta == ''){
                        this.mensajeError = 'Ingrese fecha de hasta donde desea la busqueda!';
                    }
                
                }

            
            
        },
        clean: function () {

            this.data = null;
            this.arrayPDF = [];
            this.arrayUser = [];
            this.arrayData = [];
            this.arrayDateUser = [];
            this.arrayNameAllEmpleados = [] //props a recibir en MarcacionFaltantes!
            this.arrayNameEmpledosAsistence = []
            this.arrayProp = []
            
        },
        isNull: function (param) {
            if (param != null) {
                return param.substring(11,16)
            }else {
                return ""
            }
        },
        CalculateHourWork: function (inicial, final) {
            let Out = null
            if (final == null) {
                Out = 0
            }else {
                var a = moment(final);
                var b = moment(inicial);

                Out = a.diff(b,"minutes");
            }

            return Out
        },
        ItemColor: function (inicial, final, meta) {
            let Out = null
            if (final == null) {
                Out = "0"
            }else {
                var a = moment(final);
                var b = moment(inicial);

                Out = a.diff(b,"minutes");
            }

            if (Out >= meta){return true }else{ return false}
        },
        CalculateDate: function (inicial, final, meta){
            let Out = null
            if (final == null) {
                Out = "0"
            }else {
                var a = moment(final);
                var b = moment(inicial);

                Out = a.diff(b,"minutes");
            }
            return Out+" /"+meta
        },
        isSabado: function(timeMeta, NombreDia, area){
            if(area == "Operaciones" || area == "KYC"){
                return timeMeta
            }else {
                if(NombreDia == "S"){
                    timeMeta = 240
                }
                return timeMeta
            }
        },
        BugMultiplicateUser: function () {
            var arrayOut = [];
            this.data.forEach(item=> {
            try {
                if (JSON.stringify(arrayOut[arrayOut.length-1]) !== JSON.stringify(item)) {
                arrayOut.push(item);
                }
            } catch(err) {
                arrayOut.push(item);
            }
            })

            this.arrayData = arrayOut;
        },
        BugMultiplicateUser2: function (arrayIn) {
            var arrayOut = [];
            arrayIn.forEach(item=> {
            try {
                if (JSON.stringify(arrayOut[arrayOut.length-1]) !== JSON.stringify(item)) {
                arrayOut.push(item);
                }
            } catch(err) {
                arrayOut.push(item);
            }
            })

            return arrayOut;
        },
        DatosForPDF: function (arrayIn){
            arrayIn.forEach(item =>{
                var row = {
                    "NAME_EMPLEADO_CORRECT": item.NAME_EMPLEADO_CORRECT,
                    "NAME_AREA": item.NAME_AREA,
                    "AUTH_DATE": item.AUTH_DATE,
                    "HOUR_IN": item.HOUR_IN.substring(11,16),
                    "HOUR_OUT": this.isNull(item.HOUR_OUT),
                    "WORK_DAY": this.CalculateDate(item.HOUR_IN, item.HOUR_OUT, this.isSabado(item.MIN_WORK_DAY, item.DAY_ES, item.NAME_AREA))+"min",
                    "DAY_ES": item.DAY_ES
                }

                this.arrayPDF.push(row)
            });
        },
        exportPDF: function () {
            var columns = [
                {title: "NAME", dataKey: "NAME_EMPLEADO_CORRECT"},
                {title: "AREA", dataKey: "NAME_AREA"},
                {title: "DIA", dataKey: "DAY_ES"},
                {title: "FECHA", dataKey: "AUTH_DATE"},
                {title: "ENTRADA", dataKey: "HOUR_IN"},
                {title: "SALIDA", dataKey: "HOUR_OUT"},
                {title: "MIN TRABAJADO", dataKey: "WORK_DAY"}
            ];
            var doc = new jsPDF('p', 'pt');
            doc.text('SISTEMA DE MARCACION CHIVO S.A. DE C.V.', 40, 40);
            doc.addImage("https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQtRObHZeFpWBEK4UtFBo1CYA8APnbq6im7mA&usqp=CAU",
                          'JPEG',50,50
            );
            doc.autoTable(columns, this.arrayPDF, {
                margin: {top: 150},
            });
            doc.save('TrabajadoMes.pdf');
        },
        headRows: function() {
          return [
            { NAME_EMPLEADO_CORRECT: 'NAME',
              NAME_AREA: 'AREA', 
              DAY_ES: 'DIA', 
              AUTH_DATE: 'FECHA', 
              HOUR_IN: 'ENTRADA',
              HOUR_OUT: 'SALIDA', 
              WORK_DAY: 'MIN TRABAJADO' },
          ]
        },
        exportPDF2: function () {
            // Custom style - shows how custom styles can be applied

              var doc = new jsPDF('l')
              doc.autoTable({
                head: this.headRows(),
                body: this.arrayPDF,
                margin: { top: 37 },
                tableLineColor: [44, 62, 80],
                tableLineWidth: 1,
                styles: {
                  lineColor: [44, 62, 80],
                  lineWidth: 1,
                },
                headStyles: {
                  fillColor: [241, 196, 15],
                  fontSize: 10,
                },
                bodyStyles: {
                  fillColor: [52, 73, 94],
                  textColor: 240,
                },
                alternateRowStyles: {
                  fillColor: [74, 96, 117],
                },
                columnStyles: {
                  NAME_EMPLEADO_CORRECT: {
                    fontStyle: 'bold',
                  },
                  NAME_AREA: {
                    fontStyle: 'bold',
                    //font: 'mitubachi',
                  },
                  DAY_ES: {
                    //halign: 'right',
                    fontStyle: 'bold',
                  },
                  AUTH_DATE: {
                    //halign: 'right',
                    fontStyle: 'bold',
                  },
                  HOUR_IN: {
                    //halign: 'right',
                    fontStyle: 'bold',
                  },
                  HOUR_OUT: {
                    //halign: 'right',
                    fontStyle: 'bold',
                  },
                  WORK_DAY: {
                    //halign: 'right',
                    fontStyle: 'bold',
                  },
                },
                allSectionHooks: true,
                // Use for customizing texts or styles of specific cells after they have been formatted by this plugin.
                // This hook is called just before the column width and other features are computed.
                didParseCell: function (data) {
                  
                  /* if (data.row.index === 5) {
                    data.cell.styles.fillColor = [40, 170, 100]
                  }
                  if (
                    (data.row.section === 'head' || data.row.section === 'foot') &&
                    data.column.dataKey === 'expenses'
                  ) {
                    data.cell.text = '' // Use an icon in didDrawCell instead
                  } */

                  /* if (data.column.dataKey === 'city') {
                    data.cell.styles.font = 'mitubachi'
                    if (data.row.section === 'head') {
                      data.cell.text = 'シティ'
                    }
                    if (data.row.index === 0 && data.row.section === 'body') {
                      data.cell.text = 'とうきょう'
                    }
                  } */
                },
                // Use for changing styles with jspdf functions or customize the positioning of cells or cell text
                // just before they are drawn to the page.
                willDrawCell: function (data) {
                  if (data.row.section === 'body' && data.column.dataKey === 'WORK_DAY') {
                    /* if (data.cell.raw > 750) {
                      doc.setTextColor(231, 76, 60) // Red
                    } */
                    var valueCell = data.cell.raw
                    var index = valueCell.indexOf('/')
  
                    var leng = valueCell.length 
                    var minJob = valueCell.substring(0,index)
                    var requireMin = valueCell.substring(index+1, leng)
                    //quitando paalabra min
                    var indexReq = requireMin.indexOf('m')
                    var reqMin = requireMin.substring(0, indexReq)
                    

                    console.log('min: '+ minJob)
                    console.log('requireMin: '+ reqMin)
                    
                    if (minJob == 0 ) {
                      doc.setTextColor(231, 76, 60) // Red
                    }else if( minJob > 0 && minJob <= reqMin){
                      doc.setTextColor(221, 231, 60) // Yellow
                    }else if(minJob == reqMin || minJob > reqMin){
                      doc.setTextColor(40, 170, 100) // Green
                    }                

                  }
                },
                
                /* didDrawCell: function (data) {
                  if ((data.row.section === 'head' || data.row.section === 'foot') &&
                    data.column.dataKey === 'expenses' &&
                    coinBase64Img
                  ) {
                    doc.addImage(
                      coinBase64Img,
                      'PNG',
                      data.cell.x + 5,
                      data.cell.y + 2,
                      5,
                      5
                    )
                  }
                }, */

                didDrawPage: function (data) {
                  doc.setFontSize(18)
                  doc.text('SISTEMA DE MARCACION CHIVO S.A. DE C.V.', data.settings.margin.left, 22)
                  //doc.addImage('../assets/bitcoin.png','PNG', 22, 22)
                },
              })


              //Validando el nombre que tendra el archivo.
              var nameFile = ''
              if (this.desde == this.hasta) {
                nameFile = this.desde
              }else {
                nameFile = this.desde+'||'+this.hasta
              }

              doc.save(nameFile+'.pdf');
        },
        GroupByName:  function(arrayIn, arraySave){
            arrayIn.forEach(element => {
                var name = element.NAME_EMPLEADO_CORRECT
                //var area = element.NAME_AREA
                //var row = {"NAME_EMPLEADO_CORRECT": name,"NAME_AREA": area};
                if (arraySave == null) {
                    arraySave.push(name)
                }else{
                    if (arraySave.indexOf(name) == -1) {
                        arraySave.push(name)
                    }
                }
            });

        },
        FilterEmpleadoNotAsistence: function(){
           if (this.arrayNameEmpledosAsistence != null) {
               var difference = [];
                this.arrayNameAllEmpleados.forEach(item => {

                    if (this.arrayNameEmpledosAsistence.indexOf(item) == -1) {
                         difference.push(item)
                    }

                })
                
                //console.log("Asis: "+this.arrayNameEmpledosAsistence.length)
                //console.log("Not Asis: "+this.arrayNameAllEmpleados.length)
                //console.log("Dif: "+difference.length)
                this.arrayProp = difference
           }
        },
        showFaltantes: function() {
            if (this.faltantes == false) {
                this.faltantes = true
            } else {
                this.faltantes = false
            }
        }
    },
    mounted(){
        this.LoadDates()
        this.backDataEmpleados()
    }
}
</script>