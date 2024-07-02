# 1. Promocionar un restaurante

Añadir lo siguiente a `RestaurantsController` que está en el backend, en la carpeta `controllers`

```JSX
import { Sequelize } from 'sequelize'


const promote = async function (req, res) {
  try {
    const restaurant = await Restaurant.findByPk(req.params.restaurantId)
    const otherPromotedRestaurant = await Restaurant.findOne({
      where: {
        promoted: true,
        id: { [Sequelize.Op.ne]: restaurant.id }
      }
    })
    if (otherPromotedRestaurant) {
      otherPromotedRestaurant.promoted = false
      await otherPromotedRestaurant.save()
    }
    restaurant.promoted = true
    const updatedRestaurant = await restaurant.save()
    res.json(updatedRestaurant)
  } catch (err) {
    res.status(500).send(err)
  }
}



const RestaurantController = {
  index,
  indexOwner,
  create,
  show,
  update,
  destroy,
  promote
}
```

Añadir lo siguiente a `create-restaurant` que está en el backend, en la carpeta `database/migrations`

```JSX
promoted: {
        allowNull: false,
        type: Sequelize.BOOLEAN,
        defaultValue: false
      },
```

Debajo de 

```JSX
updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: new Date()
      },
```


La carpeta `database/seeders` son para agregar datos a nuestra base de datos.


Añadir lo siguiente a `create-restaurant` que está en el backend, en la carpeta `database/migrations`

```JSX
promoted: {
        allowNull: false,
        type: Sequelize.BOOLEAN,
        defaultValue: false
      },
```


Añadir lo siguiente a `RestaurantMiddleware` que está en el backend, en la carpeta `middlewares`

```JSX
const checkNoOtherPromoted = async (req, res, next) => {
  try {
    const prom = req.body.promoted
    const isPromoted = prom ? req.body.promoted : false
    const otherPromotedRestaurant = await Restaurant.findOne({
      where: {
        promoted: true
      }
    })
    if (otherPromotedRestaurant && isPromoted) {
      return res.status(422).send('There is already some promoted restaurant')
    }
    return next()
  } catch (error) {
    return res.status(500).send(error)
  }
}
```


Añadir lo siguiente a `Restaurant` que está en el backend, en la carpeta `models`

```JSX
promoted: {
        allowNull: false,
        type: Sequelize.BOOLEAN,
        defaultValue: false
      }
```

Debajo de 

```JSX
updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: new Date()
      },
```


Añadir lo siguiente a `RestaurantRoute` que está en el backend, en la carpeta `routes`

```JSX
RestaurantMiddleware.checkNoOtherPromoted,
```

En la funcion `loadFileRoutes`

```JSX
const loadFileRoutes = function (app) {
  app.route('/restaurants')
    .get(
      RestaurantController.index)
    .post(
      isLoggedIn,
      hasRole('owner'),
      handleFilesUpload(['logo', 'heroImage'], process.env.RESTAURANTS_FOLDER),
      RestaurantValidation.create,
      handleValidation,

      RestaurantMiddleware.checkNoOtherPromoted,

      RestaurantController.create)
```

Y tambien añadir lo siguiente a `RestaurantRoute` que está en el backend, en la carpeta `routes`

```JSX
app.route('/restaurants/:restaurantId/promote')
    .patch(
      isLoggedIn,
      hasRole('owner'),
      checkEntityExists(Restaurant, 'restaurantId'),
      RestaurantMiddleware.checkRestaurantOwnership,
      RestaurantController.promote
    )
```



# 2. Frontend


Añadir lo siguiente a `RestaurantEndpoints` que está en el frontend, en la carpeta `api`

```JSX
patch

import { get, post, destroy, put, patch } from './helpers/ApiRequestsHelper'

function promote (id) {
  return patch(`restaurants/${id}/promote`)
}

promote
export { getAll, getDetail, getRestaurantCategories, create, remove, update, promote }
```



Añadir lo siguiente a `CreateRestaurant` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
<TextRegular>Is it promoted?</TextRegular>
            <Switch
                trackColor={{ false: GlobalStyles.brandSecondary, true: GlobalStyles.brandPrimary }}
                thumbColor={values.promote ? GlobalStyles.brandSecondary : '#f4f3f4'}
                // onValueChange={toggleSwitch}
                value={values.promote}
                style={styles.switch}
                onValueChange={value =>
                  setFieldValue('promote', value)
                }
              />
```



Añadir lo siguiente a `RestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
const [restaurantToBePromoted, setRestaurantToBePromoted] = useState(null)


{item.promoted &&
          <TextSemiBold textStyle={styles.promoted }>ON PROMOTION</TextSemiBold>
        }


<Pressable
            onPress={() => { setRestaurantToBePromoted(item) }}
            style={({ pressed }) => [
              {
                backgroundColor: pressed
                  ? GlobalStyles.brandGreenTap
                  : GlobalStyles.brandSuccess
              },
              styles.actionButton
            ]}>
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='star' color={'white'} size={20}/>
              <TextRegular textStyle={styles.text}>
                Promote
              </TextRegular>
            </View>
          </Pressable>
```

Debajo de 

```JSX
<Pressable
            onPress={() => { setRestaurantToBeDeleted(item) }}
            style={({ pressed }) => [
              {
                backgroundColor: pressed
                  ? GlobalStyles.brandPrimaryTap
                  : GlobalStyles.brandPrimary
              },
              styles.actionButton
            ]}>
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='delete' color={'white'} size={20}/>
              <TextRegular textStyle={styles.text}>
                Delete
              </TextRegular>
            </View>
          </Pressable>
```


Y tambien añadir lo siguiente a `RestaurantScreen` que está en el backend, en la carpeta `screens/restaurants`

```JSX
const promoteRestaurant = async (restaurant) => {
    try {
      await promote(restaurant.id)
      await fetchRestaurants()
      setRestaurantToBePromoted(null)
      showMessage({
        message: `Restaurant ${restaurant.name} succesfully promoted`,
        type: 'success',
        style: GlobalStyles.flashStyle,
        titleStyle: GlobalStyles.flashTextStyle
      })
    } catch (error) {
      setRestaurantToBePromoted(null)
      showMessage({
        message: `Restaurant ${restaurant.name} could not be promoted.`,
        type: 'error',
        style: GlobalStyles.flashStyle,
        titleStyle: GlobalStyles.flashTextStyle
      })
    }
  }
```




Todo esto dentro de `RestaurantScreen` que está en el backend, en la carpeta `screens/restaurants` dentro de la funcion:

```JSX
export default function RestaurantsScreen ({ navigation, route }) {
```

Y tambien añadir lo siguiente a `RestaurantScreen` que está en el backend, en la carpeta `screens/restaurants` dentro del return de la funcion anterior detrás del `FlatList`

```JSX
<DeleteModal
  isVisible={restaurantToBeDeleted !== null}
  onCancel={() => setRestaurantToBeDeleted(null)}
  onConfirm={() => removeRestaurant(restaurantToBeDeleted)}>
    <TextRegular>The products of this restaurant will be deleted as well</TextRegular>
    <TextRegular>If the restaurant has orders, it cannot be deleted.</TextRegular>
</DeleteModal>

<DeleteModal
  isVisible={restaurantToBePromoted !== null}
  onCancel={() => setRestaurantToBePromoted(null)}
  onConfirm={() => promoteRestaurant(restaurantToBeDeleted)}>
    <TextRegular>The products of this restaurant will be promoted as well</TextRegular>
    <TextRegular>If the restaurant has orders, it cannot be promoted.</TextRegular>
</DeleteModal>
```
