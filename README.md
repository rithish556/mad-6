package com.example.locationapp

import android.Manifest
import android.content.pm.PackageManager
import android.location.Address
import android.location.Geocoder
import android.location.Location
import android.os.Bundle
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationServices
import java.io.IOException
import java.util.*

class MainActivity : AppCompatActivity() {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private val LOCATION_REQUEST_CODE = 100

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

        // Start fetching location
        getLocation()
    }

    private fun getLocation() {
        if (ActivityCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED &&
            ActivityCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_COARSE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
                LOCATION_REQUEST_CODE
            )
            return
        }

        fusedLocationClient.lastLocation.addOnSuccessListener { location: Location? ->
            location?.let {
                val latitude = it.latitude
                val longitude = it.longitude
                val address = getAddressFromLocation(latitude, longitude)
                val message = "Latitude: $latitude\nLongitude: $longitude\n\nAddress:\n$address"
                showAlert(message)
            } ?: showAlert("Location not available")
        }
    }

    private fun getAddressFromLocation(lat: Double, lon: Double): String {
        val geocoder = Geocoder(this, Locale.getDefault())
        return try {
            val addresses: List<Address> =
                geocoder.getFromLocation(lat, lon, 1) ?: emptyList()
            if (addresses.isNotEmpty()) {
                val address: Address = addresses[0]
                val sb = StringBuilder()
                sb.append(address.getAddressLine(0)).append("\n")
                sb.append(address.locality).append("\n")
                sb.append(address.adminArea).append("\n")
                sb.append(address.countryName).append("\n")
                sb.append("Postal Code: ").append(address.postalCode).append("\n")
                sb.toString()
            } else {
                "No address found"
            }
        } catch (e: IOException) {
            e.printStackTrace()
            "Unable to get address"
        }
    }

    private fun showAlert(message: String) {
        AlertDialog.Builder(this)
            .setTitle("Current Location Info")
            .setMessage(message)
            .setPositiveButton("OK", null)
            .show()
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == LOCATION_REQUEST_CODE) {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                getLocation()
            } else {
                showAlert("Permission denied. Cannot fetch location.")
            }
        }
    }
}
