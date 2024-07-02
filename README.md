# 1. Promocionar un restaurante

Añadir lo siguiente a `ProductController` que está en el backend, en la carpeta `controllers`

```JSX
newProduct.basePrice = newProduct.price

```
Dentro de 
```JSX
const create = async function (req, res) {
  let newProduct = Product.build(req.body)
  try {

    // Solution: basePrice updated from price property (given in the student's base project)
    newProduct.basePrice = newProduct.price

    newProduct = await newProduct.save()
    res.json(newProduct)
  } catch (err) {
    res.status(500).send(err)
  }
}
```

Añadir lo siguiente a `ProductController` que está en el backend, en la carpeta `controllers`

```JSX
req.body.basePrice = req.body.price

```
Dentro de 
```JSX
const update = async function (req, res) {
  try {
    // Solution: basePrice updated from new price property (given in the student's base project)
    req.body.basePrice = req.body.price
    await Product.update(req.body, { where: { id: req.params.productId } })
    let updatedProduct = await Product.findByPk(req.params.productId)
    res.json(updatedProduct)
  } catch (err) {
    res.status(500).send(err)
  }
}
```

Añadir lo siguiente a `RestaurantController` que está en el backend, en la carpeta `controllers`

```JSX
sequelizeSession
import { sequelizeSession, Restaurant, Product, RestaurantCategory, ProductCategory } from '../models/models.js'
import Sequelize from 'sequelize'

```
Y añadir tambien lo siguiente a `RestaurantController` que está en el backend, en la carpeta `controllers`
```JSX
const update = async function (req, res) {
  try {
    // Solution: not explicitly requested, but the use of a transaction is valued
    const transaction = await sequelizeSession.transaction()
    await Restaurant.update(req.body, { where: { id: req.params.restaurantId } }, transaction)

    const productsToBeUpdated = await Product.findAll({
      where: {
        restaurantId: req.params.restaurantId
      }
    });

    for (const product of productsToBeUpdated) {
      const newPrice = product.basePrice + product.basePrice * (req.body.percentage / 100);
      await product.update({ price: newPrice }, transaction);
    }

    await transaction.commit()

    const updatedRestaurant = await Restaurant.findByPk(req.params.restaurantId)

    res.json(updatedRestaurant)
  } catch (err) {
    res.status(500).send(err)
  }
}
```



Añadir lo siguiente a `RestaurantValidation` que está en el backend, en la carpeta `controllers/validations`

```JSX
check('percentage').exists().isFloat({ min: -5, max: 5 }).toFloat(),
```

Dentro de 

```JSX
const update = [
  check('name').exists().isString().isLength({ min: 1, max: 255 }).trim(),
  check('description').optional({ nullable: true, checkFalsy: true }).isString().trim(),
  check('address').exists().isString().isLength({ min: 1, max: 255 }).trim(),
  check('postalCode').exists().isString().isLength({ min: 1, max: 255 }),

  // Solution: exists validation depends on the implementation of the frontend. If percentage can be
  // empty and no initial value is set in front, this property should be optional.
  check('percentage').exists().isFloat({ min: -5, max: 5 }).toFloat(),

  check('url').optional({ nullable: true, checkFalsy: true }).isString().isURL().trim(),
  check('shippingCosts').exists().isFloat({ min: 0 }).toFloat(),
  check('email').optional({ nullable: true, checkFalsy: true }).isString().isEmail().trim(),
  check('phone').optional({ nullable: true, checkFalsy: true }).isString().isLength({ min: 1, max: 255 }).trim(),
  check('restaurantCategoryId').exists({ checkNull: true }).isInt({ min: 1 }).toInt(),
  check('userId').not().exists(),
  check('heroImage').custom((value, { req }) => {
    return checkFileIsImage(req, 'heroImage')
  }).withMessage('Please upload an image with format (jpeg, png).'),
  check('heroImage').custom((value, { req }) => {
    return checkFileMaxSize(req, 'heroImage', maxFileSize)
  }).withMessage('Maximum file size of ' + maxFileSize / 1000000 + 'MB'),
  check('logo').custom((value, { req }) => {
    return checkFileIsImage(req, 'logo')
  }).withMessage('Please upload an image with format (jpeg, png).'),
  check('logo').custom((value, { req }) => {
    return checkFileMaxSize(req, 'logo', maxFileSize)
  }).withMessage('Maximum file size of ' + maxFileSize / 1000000 + 'MB'),

]
```

Añadir lo siguiente a `create-restaurant` que está en el backend, en la carpeta `database/migrations`

```JSX
percentage: {
        type: Sequelize.DOUBLE,
        defaultValue: 0.0
      },
```

Debajo de 

```JSX
 status: {
        type: Sequelize.ENUM,
        values: [
          'online',
          'offline',
          'closed',
          'temporarily closed'
        ],
        defaultValue: 'offline'
      },
```


Añadir a la carpeta `database/seeders` para agregar datos a nuestra base de datos en `ProductSeeders`.

```JSX
 basePrice: 2.5,
```

Dentro de 

```JSX
 { name: 'Ensaladilla', description: 'Tuna salad with mayonnaise', price: 2.5, basePrice: 2.5, image: process.env.PRODUCTS_FOLDER + '/ensaladilla.jpeg', order: 1, availability: true, restaurantId: 1, productCategoryId: 1 },
        { name: 'Olives', description: 'Home made', price: 1.5, basePrice: 1.5, image: process.env.PRODUCTS_FOLDER + '/aceitunas.jpeg', order: 2, availability: true, restaurantId: 1, productCategoryId: 1 },
```
Asi en todos los datos




Añadir lo siguiente a `Product` que está en el backend, en la carpeta `models`

```JSX
basePrice: DataTypes.DOUBLE,
```

Dentro de:

```JSX
Product.init({
    name: DataTypes.STRING,
    description: DataTypes.STRING,
    price: DataTypes.DOUBLE,

    // Solution
    basePrice: DataTypes.DOUBLE,

    image: DataTypes.STRING,
    order: DataTypes.INTEGER,
    availability: DataTypes.BOOLEAN,
    restaurantId: DataTypes.INTEGER,
    productCategoryId: DataTypes.INTEGER
  }, {
    sequelize,
    modelName: 'Product'
  })
```


Añadir lo siguiente a `Restaurant` que está en el backend, en la carpeta `models`

```JSX
percentage: {
      type: DataTypes.DOUBLE,
      defaultValue: 0.0
    },
```
Dentro de:

```JSX
Restaurant.init({
    name: {
      allowNull: false,
      type: DataTypes.STRING
    },
    description: DataTypes.TEXT,
    address: {
      allowNull: false,
      type: DataTypes.STRING
    },
    postalCode: {
      allowNull: false,
      type: DataTypes.STRING
    },
    url: DataTypes.STRING,
    shippingCosts: {
      allowNull: false,
      type: DataTypes.DOUBLE
    },
    averageServiceMinutes: DataTypes.DOUBLE,
    email: DataTypes.STRING,
    phone: DataTypes.STRING,
    logo: DataTypes.STRING,
    heroImage: DataTypes.STRING,
    status: {
      type: DataTypes.ENUM,
      values: [
        'online',
        'offline',
        'closed',
        'temporarily closed'
      ]
    },


    // Solution
    percentage: {
      type: DataTypes.DOUBLE,
      defaultValue: 0.0
    },


    restaurantCategoryId: {
      allowNull: false,
      type: DataTypes.INTEGER
    },
    userId: {
      allowNull: false,
      type: DataTypes.INTEGER
    },
    createdAt: {
      allowNull: false,
      type: DataTypes.DATE,
      defaultValue: new Date()
    },
    updatedAt: {
      allowNull: false,
      type: DataTypes.DATE,
      defaultValue: new Date()
    }
  }, {
    sequelize,
    modelName: 'Restaurant'
  })
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
