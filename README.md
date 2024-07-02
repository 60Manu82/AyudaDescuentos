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


Crear lo siguiente a `ConfirmationModal` que está en el frontend, en la carpeta `Components`

```JSX
// This file has been created for solution

import React from 'react'
import { Modal, Pressable, StyleSheet, View } from 'react-native'
import { MaterialCommunityIcons } from '@expo/vector-icons'
import TextSemiBold from './TextSemibold'
import * as GlobalStyles from '../styles/GlobalStyles'
import TextRegular from './TextRegular'
export default function ConfirmationModal(props) {
  return (
    <Modal
      presentationStyle='overFullScreen'
      animationType='slide'
      transparent={true}
      visible={props.isVisible}
      onRequestClose={props.onCancel}>
      <View style={styles.centeredView}>
        <View style={styles.modalView}>
          <TextSemiBold textStyle={{ fontSize: 15 }}>A corrective percentage will be applied to the price of this restaurant's products</TextSemiBold>
          {props.children}
          <Pressable
            onPress={props.onCancel}
            style={({ pressed }) => [
              {
                backgroundColor: pressed
                  ? GlobalStyles.brandBlueTap
                  : GlobalStyles.brandBlue
              },
              styles.actionButton
            ]}>
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='close' color={'white'} size={20} />
              <TextRegular textStyle={styles.text}>
                Cancel
              </TextRegular>
            </View>
          </Pressable>
          <Pressable
            onPress={props.onConfirm}
            style={({ pressed }) => [
              {
                backgroundColor: pressed
                  ? GlobalStyles.brandPrimaryTap
                  : GlobalStyles.brandPrimary
              },
              styles.actionButton
            ]}>
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='check-outline' color={'white'} size={20} />
              <TextRegular textStyle={styles.text}>
                Confirm
              </TextRegular>
            </View>
          </Pressable>
        </View>
      </View>
    </Modal>
  )
}

const styles = StyleSheet.create({
  centeredView: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    marginTop: 22
  },
  modalView: {
    margin: 20,
    backgroundColor: 'white',
    borderRadius: 20,
    padding: 35,
    alignItems: 'center',
    shadowOffset: {
      width: 0,
      height: 2
    },
    shadowOpacity: 0.75,
    shadowRadius: 4,
    elevation: 5,
    width: '90%'
  },
  actionButton: {
    borderRadius: 8,
    height: 40,
    marginTop: 12,
    margin: '1%',
    padding: 10,
    alignSelf: 'center',
    flexDirection: 'column',
    width: '50%'
  },
  text: {
    fontSize: 16,
    color: 'white',
    alignSelf: 'center',
    marginLeft: 5
  }
})
```



Añadir lo siguiente a `CreateRestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
percentage
import { create, getRestaurantCategories, percentage } from '../../api/RestaurantEndpoints'

percentage: 0
const initialRestaurantValues = { name: null, description: null, address: null, postalCode: null, url: null, shippingCosts: null, email: null, phone: null, restaurantCategoryId: null, percentage: 0 }
```

Añadir lo siguiente a `EditRestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
import ConfirmationModal from '../../components/ConfirmationModal'
import { MaterialCommunityIcons } from '@expo/vector-icons'
import TextSemiBold from '../../components/TextSemibold'
```

Y añadir tambien lo siguiente a `EditRestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
percentage: 0
const [initialRestaurantValues, setInitialRestaurantValues] = useState({ name: null, description: null, address: null, postalCode: null, url: null, shippingCosts: null, email: null, phone: null, restaurantCategoryId: null, logo: null, heroImage: null, percentage: 0 })
  const [percentageShowDialog, setPercentageShowDialog] = useState(false)

Dentro del yup
 percentage: yup
      .number()
      .max(5)
      .min(-5)


const updateRestaurant = async (values) => {
    setBackendErrors([])

    // Solution
    if (values.percentage != 0 && !percentageShowDialog) {
      setPercentageShowDialog(true)
    } else {
      // Solution
      setPercentageShowDialog(false)

      try {
        const updatedRestaurant = await update(restaurant.id, values)
        showMessage({
          message: `Restaurant ${updatedRestaurant.name} succesfully updated`,
          type: 'success',
          style: GlobalStyles.flashStyle,
          titleStyle: GlobalStyles.flashTextStyle
        })
        navigation.navigate('RestaurantsScreen', { dirty: true })
      } catch (error) {
        console.log(error)
        setBackendErrors(error.errors)
      }
    }
  }

```

Y añadir tambien lo siguiente a `EditRestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants` dentro del formik
```JSX
<View style={{ flexDirection: 'row', justifyContent: 'center', alignItems: 'center', marginTop: 20, marginBottom: 10 }} >
                <Pressable onPress={() => {
                  let newPercentage = values.percentage + 0.5
                  if (newPercentage < 5)
                    setFieldValue('percentage', newPercentage)
                }}>
                  <MaterialCommunityIcons
                    name={'arrow-up-circle'}
                    color={GlobalStyles.brandSecondaryTap}
                    size={40}
                  />
                </Pressable>

                <TextSemiBold>Porcentaje actual: <TextSemiBold textStyle={{ color: GlobalStyles.brandPrimary }}>{values.percentage.toFixed(1)}%</TextSemiBold></TextSemiBold>

                <Pressable onPress={() => {
                  let newPercentage = values.percentage - 0.5
                  if (newPercentage > -5)
                    setFieldValue('percentage', newPercentage)
                }}>
                  <MaterialCommunityIcons
                    name={'arrow-down-circle'}
                    color={GlobalStyles.brandSecondaryTap}
                    size={40}
                  />
                </Pressable>
              </View>



Al final del formik

<ConfirmationModal
            isVisible={percentageShowDialog}
            onCancel={() => setPercentageShowDialog(false)}
            onConfirm={() => updateRestaurant(values)}>
          </ConfirmationModal>
```




Añadir lo siguiente a `RestaurantScreen` que está en el frontend, en la carpeta `screens/restaurants`

```JSX
{item.percentage != 0 && <View style={{ flexDirection: 'row', justifyContent: 'space-between', alignItems: 'flex-end' }} >
          <TextSemiBold textStyle={{ color: item.percentage > 0 ? 'red' : 'green' }}>{item.percentage > 0 ? '¡Incremento de precios aplicados!' : '¡Descuentos aplicados!'}</TextSemiBold>
        </View>
        }
```

Debajo de 

```JSX
<TextRegular numberOfLines={2}>{item.description}</TextRegular>
        {item.averageServiceMinutes !== null &&
          <TextSemiBold>Avg. service time: <TextSemiBold textStyle={{ color: GlobalStyles.brandPrimary }}>{item.averageServiceMinutes} min.</TextSemiBold></TextSemiBold>
        }

        <View style={{ flexDirection: 'row', justifyContent: 'space-between', alignItems: 'flex-end' }} >
          <TextSemiBold>Shipping: <TextSemiBold textStyle={{ color: GlobalStyles.brandPrimary }}>{item.shippingCosts.toFixed(2)}€</TextSemiBold></TextSemiBold>
        </View>
```

Encima de 

```JSX
<View style={styles.actionButtonsContainer}>
          <Pressable
            onPress={() => navigation.navigate('EditRestaurantScreen', { id: item.id })}
            style={({ pressed }) => [
              {
                backgroundColor: pressed ? GlobalStyles.brandBlueTap : GlobalStyles.brandBlue
              },
              styles.actionButton
            ]}
          >
            <View style={[{ flex: 1, flexDirection: 'row', justifyContent: 'center' }]}>
              <MaterialCommunityIcons name='pencil' color={'white'} size={20} />
              <TextRegular textStyle={styles.text}>Edit</TextRegular>
            </View>
          </Pressable>
```
