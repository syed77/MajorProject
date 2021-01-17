package com.communication.majorproject.ui.slideshow;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.Spinner;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProviders;

import com.android.volley.AuthFailureError;
import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.StringRequest;
import com.android.volley.toolbox.Volley;
import com.communication.majorproject.DrawerActivity;
import com.communication.majorproject.R;
import com.google.gson.Gson;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static android.app.Activity.RESULT_OK;
import static android.content.ContentValues.TAG;

public class SlideshowFragment extends Fragment implements View.OnClickListener{

    private SlideshowViewModel slideshowViewModel;
    Spinner spinner;
    Button uploadButton;
    final int FILE_SELECT_CODE = 0;
    String path;


    public View onCreateView(@NonNull LayoutInflater inflater,
                             ViewGroup container, Bundle savedInstanceState) {
        slideshowViewModel =
                ViewModelProviders.of(this).get(SlideshowViewModel.class);
        View root = inflater.inflate(R.layout.fragment_slideshow, container, false);

        return root;
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        uploadButton = getView().findViewById(R.id.uploadBtn);
        uploadButton.setOnClickListener(this);

        final List<String> em = new ArrayList<>();
        spinner = (Spinner) getView().findViewById(R.id.spin_id);
        final ArrayAdapter aa = new ArrayAdapter(getContext(),android.R.layout.simple_spinner_item, em);
        aa.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        spinner.setAdapter(aa);

        RequestQueue requestQueue = Volley.newRequestQueue(getContext());
        String Url = getString(R.string.base_url)+"db_access.php?";


        StringRequest stringRequest = new StringRequest(Request.Method.POST, Url+"email=syedmhd77@gmail.com", new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
//                Toast.makeText(getContext(), response, Toast.LENGTH_LONG).show();

                Log.i("response",response);
                try {
                    JSONObject obj = new JSONObject(response);
                    JSONArray array =  obj.getJSONArray("email");
                    for (int i = 0 ; i<array.length();i++) {
                        aa.add(array.getString(i));
                    }

                } catch (JSONException e) {
                    e.printStackTrace();
                }

            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("VOLLEY", error.toString());
            }
        })

        {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                Map<String,String> params = new HashMap<>();
                params.put("","");
                return super.getParams();
            }
        };

        requestQueue.add(stringRequest);
    }

    @Override
    public void onResume() {
        super.onResume();
        ((DrawerActivity) getActivity()).getSupportActionBar().setTitle("Update Info");
    }

    @Override
    public void onClick(View v) {

        showFileChooser();
    }


    public void rdb_insert(String path,String Url)
    {
        RequestQueue requestQueue = Volley.newRequestQueue(getContext());
        File file = new File(path);
        String output = "";
        try {
            BufferedReader br = new BufferedReader(new FileReader(file));
            String st="";
            while ((st = br.readLine()) != null)
                output+=st;

            final JSONObject jsonObject = new JSONObject(output);

            Log.i("json", String.valueOf(jsonObject.getInt("pid")));
            final String s = spinner.getSelectedItem().toString();
            jsonObject.put("pid",s);

            Gson gson = new Gson();
            final Map<String,String> map = gson.fromJson(output, Map.class);


            JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(Request.Method.POST, Url, jsonObject, new Response.Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject response) {
                    Toast.makeText(getContext(),response.toString(),Toast.LENGTH_LONG).show();
                    Log.i("msg recvd",response.toString());
                }
            }, new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    Toast.makeText(getContext(),"error is "+error.toString(),Toast.LENGTH_LONG).show();
                    Log.e("err",error.toString());
                    error.printStackTrace();
                }
            })
            {
                @Override
                protected Map<String, String> getParams() throws AuthFailureError {
                    map.put("pid","arfathsyed@gmail.com");
                    return map;
                }
            };

            requestQueue.add(jsonObjectRequest);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }



    public void showFileChooser() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("*/*");
        intent.addCategory(Intent.CATEGORY_OPENABLE);

        try {
            startActivityForResult(
                    Intent.createChooser(intent, "Select a File to Upload"),
                    FILE_SELECT_CODE);
        } catch (android.content.ActivityNotFoundException ex) {
            // Potentially direct the user to the Market with a Dialog
            Toast.makeText(getContext(), "Please install a File Manager.",
                    Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
            case FILE_SELECT_CODE:
                if (resultCode == RESULT_OK) {
                    // Get the Uri of the selected file
                    Uri uri = data.getData();
                    Log.d(TAG, "File Uri: " + uri.toString());

                    if (uri.toString().contains("m_data"))
                        path = "/storage/emulated/0/project/m_data.json";
                    else
                        path = "/storage/emulated/0/project/b_data.json";

                    String url = getString(R.string.base_url)+"db_insert_r.php?";
                    rdb_insert(path,url);
                }
                break;
        }
        super.onActivityResult(requestCode, resultCode, data);
    }


}
