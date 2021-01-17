package com.communication.majorproject.ui.delete;

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
import com.android.volley.toolbox.StringRequest;
import com.android.volley.toolbox.Volley;
import com.communication.majorproject.DrawerActivity;
import com.communication.majorproject.R;
import com.communication.majorproject.ui.gallery.GalleryViewModel;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class DeleteFragment extends Fragment implements View.OnClickListener {

    Button delBttn;
    Spinner spinner;

    private DeleteViewModel deleteViewModel;
    public View onCreateView(@NonNull LayoutInflater inflater,
                             ViewGroup container, Bundle savedInstanceState) {
        deleteViewModel =
                ViewModelProviders.of(this).get(DeleteViewModel.class);
        View root = inflater.inflate(R.layout.fragment_delete, container, false);

        return root;
    }

    @Override
    public void onResume() {
        super.onResume();
        // Set title bar
        ((DrawerActivity) getActivity()).getSupportActionBar().setTitle("Delete Patient");
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        delBttn = getView().findViewById(R.id.deleteBtn);
        delBttn.setOnClickListener(this);

        final List<String> em = new ArrayList<>();
        spinner = (Spinner) getView().findViewById(R.id.spin_id_del);
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
//                Map<String,String> params = new HashMap<>();
//                params.put("","");
                return super.getParams();
            }
        };

        requestQueue.add(stringRequest);
    }

    @Override
    public void onClick(View v) {
        RequestQueue requestQueue = Volley.newRequestQueue(getContext());
        String Url = getString(R.string.base_url)+"db_delete.php?";

        Url +="email=syedmhd77@gmail.com&del_em="+spinner.getSelectedItem().toString();
        StringRequest stringRequest = new StringRequest(Request.Method.GET, Url,new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                Toast.makeText(getContext(),response.toString(),Toast.LENGTH_LONG).show();
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                error.printStackTrace();
            }
        });

        requestQueue.add(stringRequest);
    }
}